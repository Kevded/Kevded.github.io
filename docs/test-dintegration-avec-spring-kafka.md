---
description: 'Exemple de test d''intégration avec spring-kafka, spring-kafka-test.'
---

# Test d'intégration avec Spring Boot et Kafka

Date de publication : 26/04/2020

## Dépendences

{% hint style="info" %}
 spring-kafka version 2.4.5 \(packagé avec spring boot 2.2.6\)

spring boot version 2.2.6
{% endhint %}

{% code title="pom.xml" %}
```markup
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>integrationtestspringkafka</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>integration-test-spring-kafka-with-embedded-kafka-consumerService-and-producer</name>
    <description>Integration Test with spring kafka Embedded Kafka Consumer Producer</description>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-core</artifactId>
            <version>2.10.2</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.10.2</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-annotations</artifactId>
            <version>2.10.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
            <version>1.4.200</version>
        </dependency>
        <!-- Test -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.3.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.3.2</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <!--Exclude junit4-->
            <exclusions>
                <exclusion>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                </exclusion>
				<exclusion>
					<groupId>org.junit.vintage</groupId>
					<artifactId>junit-vintage-engine</artifactId>
				</exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.awaitility</groupId>
            <artifactId>awaitility</artifactId>
            <version>4.0.2</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.22.2</version>
            </plugin>
        </plugins>
    </build>

</project>
```
{% endcode %}

## Configuration

{% tabs %}
{% tab title="ConsumerOfExampleDTOConfig.java" %}
```java
package com.example.integrationtestspringkafka.config;

import com.example.integrationtestspringkafka.dto.ExampleDTO;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.config.ConcurrentKafkaListenerContainerFactory;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class ConsumerOfExampleDTOConfig {
    @Value("${kafka.bootstrap-servers}")
    private String bootstrapServers;

    @Bean
    ConcurrentKafkaListenerContainerFactory<String, ExampleDTO> kafkaListenerContainerFactory() {
        ConcurrentKafkaListenerContainerFactory<String, ExampleDTO> factory = new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(consumerFactory());
        return factory;
    }

    @Bean
    public ConsumerFactory<String, ExampleDTO> consumerFactory() {
        Map<String, Object> props = new HashMap<>();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
        //  ensures the new consumer group gets the messages we sent, because the container might start after the sends have completed.
        // see https://docs.spring.io/spring-kafka/docs/2.4.5.RELEASE/reference/html/
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        return new DefaultKafkaConsumerFactory<>(props, new StringDeserializer(), new JsonDeserializer<>(ExampleDTO.class, false));
    }

}
```
{% endtab %}

{% tab title="ProducerOfExampleDTOConfig.java" %}
```java
package com.example.integrationtestspringkafka.config;

import com.example.integrationtestspringkafka.dto.ExampleDTO;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.common.serialization.StringSerializer;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.kafka.annotation.EnableKafka;
import org.springframework.kafka.core.DefaultKafkaProducerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.kafka.core.ProducerFactory;
import org.springframework.kafka.support.serializer.JsonSerializer;

import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableKafka
public class ProducerOfExampleDTOConfig {
	  @Value("${kafka.bootstrap-servers}")
	  private String bootstrapServers;

	  @Bean
	  public ProducerFactory<String, ExampleDTO> producerFactory() {
		    Map<String, Object> props = new HashMap<>();

		    props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, bootstrapServers);
		    props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
		    props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, JsonSerializer.class);

	    return new DefaultKafkaProducerFactory<>(props);
	  }

	  @Bean
	  public KafkaTemplate<String, ExampleDTO> kafkaTemplate() {
	    return new KafkaTemplate<>(producerFactory());
	  }
}
```
{% endtab %}

{% tab title="DatabaseConfig .java" %}
```java
package com.example.integrationtestspringkafka.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;

@Configuration
@EnableJpaRepositories("com.example.integrationtestspringkafka.repository")
public class DatabaseConfig {
}
```
{% endtab %}
{% endtabs %}

{% code title="application.properties" %}
```groovy
kafka.bootstrap-servers=localhost:8080

# database
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
#spring.datasource.username=test
#spring.datasource.password=test
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# jpa
spring.jpa.hibernate.ddl-auto=create-drop
```
{% endcode %}

### DTO, Entity, Repository

{% tabs %}
{% tab title="ExampleDTO .java" %}
```java
package com.example.integrationtestspringkafka.dto;

public class ExampleDTO {

    private String name;
    private String description;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    @Override
    public String toString() {
        return "ExampleDTO [name=" + name + ", description=" + description + "]";
    }

}
```
{% endtab %}

{% tab title="ExampleEntity.java" %}
```java
package com.example.integrationtestspringkafka.entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;

@Entity
public class ExampleEntity {

    @Id
    @GeneratedValue(strategy= GenerationType.AUTO)
    private Long id;

    private String name;

    private String description;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    @Override
    public String toString() {
        return "ExampleEntity{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", description='" + description + '\'' +
                '}';
    }
}
```
{% endtab %}

{% tab title="ExampleRepositoy.java" %}
```java
package com.example.integrationtestspringkafka.repository;

import com.example.integrationtestspringkafka.entity.ExampleEntity;
import org.springframework.data.repository.CrudRepository;

import java.util.List;

public interface ExampleRepository extends CrudRepository<ExampleEntity, Long> {
    List<ExampleEntity> findAll();
}
```
{% endtab %}
{% endtabs %}

### Services Consumer et Producer

Le `ConsumerService` écoute sur le topic `TOPIC_EXAMPLE` et s'attend à recevoir un objet `ExampleDTO`_._ Chaque `ExampleDTO` reçu sera d'abord converti en `ExampleEntity` puis sauvegarder dans la base de données.

Le `ProducerService` sert à publier des objets `ExampleDTO` sur le topic `TOPIC_EXAMPLE_EXTERNE`.

{% tabs %}
{% tab title="ConsumerService .java" %}
```java
package com.example.integrationtestspringkafka.service;

import com.example.integrationtestspringkafka.dto.ExampleDTO;
import com.example.integrationtestspringkafka.entity.ExampleEntity;
import com.example.integrationtestspringkafka.repository.ExampleRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.annotation.KafkaListener;
import org.springframework.stereotype.Service;

@Service
public class ConsumerService {
    Logger log = LoggerFactory.getLogger(ConsumerService.class);
    private ExampleRepository exampleRepository;

    ConsumerService(ExampleRepository exampleRepository) {
        this.exampleRepository = exampleRepository;
    }

    /**
     * Consume ExampleDTO on topic : TOPIC_EXAMPLE
     * Then save it in database.
     *
     * @param exampleDTO {@link ExampleDTO}
     */
    @KafkaListener(topics = "TOPIC_EXAMPLE", groupId = "consumer_example_dto")
    public void consumeExampleDTO(ExampleDTO exampleDTO)  {
        log.info("Received from topic=TOPIC_EXAMPLE ExampleDTO={}", exampleDTO);
        exampleRepository.save(convertToExampleEntity(exampleDTO));
        log.info("saved in database {}", exampleDTO);
    }

    /**
     * In Java world you should use an Mapper, or an dedicated service to do this.
     */
    public ExampleEntity convertToExampleEntity(ExampleDTO exampleDTO) {
        ExampleEntity exampleEntity = new ExampleEntity();
        exampleEntity.setDescription(exampleDTO.getDescription());
        exampleEntity.setName(exampleDTO.getName());
        return exampleEntity;
    }
}
```
{% endtab %}

{% tab title="ProducerService .java" %}
```java
package com.example.integrationtestspringkafka.service;

import com.example.integrationtestspringkafka.dto.ExampleDTO;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.kafka.core.KafkaTemplate;
import org.springframework.stereotype.Service;

@Service
public class ProducerService {
    Logger log = LoggerFactory.getLogger(ProducerService.class);

    private String topic = "TOPIC_EXAMPLE_EXTERNE";

    private KafkaTemplate<String, ExampleDTO> kafkaTemplate;

    ProducerService(KafkaTemplate kafkaTemplate) {
        this.kafkaTemplate = kafkaTemplate;
    }

    /**
     * Send ExampleDTO to an external topic : TOPIC_EXAMPLE_EXTERNE.
     *
     * @param exampleDTO
     */
    public void send(ExampleDTO exampleDTO) {
        log.info("send to topic={} ExampleDTO={}", topic, exampleDTO);
        kafkaTemplate.send(topic, exampleDTO);
    }
}
```
{% endtab %}
{% endtabs %}

## Test d'intégration

### Test configuration

{% code title="test/resources/application.properties" %}
```java
# take automaticaly the generated address & port that embedded kafka has started
kafka.bootstrap-servers=${spring.embedded.kafka.brokers}

# database
spring.datasource.url=jdbc:h2:mem:testdb
spring.datasource.driverClassName=org.h2.Driver
#spring.datasource.username=test
#spring.datasource.password=test
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect

# jpa
spring.jpa.hibernate.ddl-auto=create-drop
```
{% endcode %}

### Test d'intégration Consumer

Pour vérifier que notre `ConsumerService` fonctionne correctement. On crée un producer qui va servir dans uniquement dans le cadre du test à publié un objet `ExampleDTO` sur le topic `TOPIC_EXAMPLE`. On sait que notre `ConsumerService` est responsable de sauvegarder dans la base de données. On va donc vérifier en base si l'`ExampleDTO` envoyé précédemment à bien été enregistré. Pour vérifier de façon asynchrone on utilise la librairie `Awaitility`.

{% code title="ConsumerServiceIntegrationTest.java" %}
```java
package com.example.integrationtestspringkafka.service;

import com.example.integrationtestspringkafka.dto.ExampleDTO;
import com.example.integrationtestspringkafka.entity.ExampleEntity;
import com.example.integrationtestspringkafka.repository.ExampleRepository;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.Producer;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringSerializer;
import org.awaitility.Durations;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.support.serializer.JsonSerializer;
import org.springframework.kafka.test.EmbeddedKafkaBroker;
import org.springframework.kafka.test.context.EmbeddedKafka;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.util.Map;
import java.util.concurrent.ExecutionException;

import static org.awaitility.Awaitility.await;
import static org.junit.jupiter.api.Assertions.assertEquals;

@ExtendWith(SpringExtension.class)
@SpringBootTest
@DirtiesContext
@EmbeddedKafka(topics = {"TOPIC_EXAMPLE", "TOPIC_EXAMPLE_EXTERNE"})
public class ConsumerServiceIntegrationTest {
    Logger log = LoggerFactory.getLogger(ConsumerServiceIntegrationTest.class);

    private static final String TOPIC_EXAMPLE = "TOPIC_EXAMPLE";

    @Autowired
    private EmbeddedKafkaBroker embeddedKafkaBroker;
    @Autowired

    private ConsumerService consumerService;
    @Autowired
    private ExampleRepository exampleRepository;

    public ExampleDTO mockExampleDTO(String name, String description) {
        ExampleDTO exampleDTO = new ExampleDTO();
        exampleDTO.setDescription(description);
        exampleDTO.setName(name);
        return exampleDTO;
    }

    /**
     * We verify the output in the topic. But aslo in the database.
     */
    @Test
    public void itShould_ConsumeCorrectExampleDTO_from_TOPIC_EXAMPLE_and_should_saveCorrectExampleEntity() throws ExecutionException, InterruptedException {
        // GIVEN
        ExampleDTO exampleDTO = mockExampleDTO("Un nom 2", "Une description 2");
        // simulation consumer
        Map<String, Object> producerProps = KafkaTestUtils.producerProps(embeddedKafkaBroker.getBrokersAsString());
        Producer<String, ExampleDTO> producerTest = new KafkaProducer(producerProps, new StringSerializer(), new JsonSerializer<ExampleDTO>());
        // Or
        // ProducerFactory producerFactory = new DefaultKafkaProducerFactory<String, ExampleDTO>(producerProps, new StringSerializer(), new JsonSerializer<ExampleDTO>());
        // Producer<String, ExampleDTO> producerTest = producerFactory.createProducer();
        // Or
        // ProducerRecord<String, ExampleDTO> producerRecord = new ProducerRecord<String, ExampleDTO>(TOPIC_EXAMPLE, "key", exampleDTO);
        // KafkaTemplate<String, ExampleDTO> template = new KafkaTemplate<>(producerFactory);
        // template.setDefaultTopic(TOPIC_EXAMPLE);
        // template.send(producerRecord);
        // WHEN
        producerTest.send(new ProducerRecord(TOPIC_EXAMPLE, "", exampleDTO));
        // THEN
        // we must have 1 entity inserted
        // We cannot predict when the insertion into the database will occur. So we wait until the value is present. Thank to Awaitility.
        await().atMost(Durations.TEN_SECONDS).untilAsserted(() -> {
            var exampleEntityList = exampleRepository.findAll();
            assertEquals(1, exampleEntityList.size());
            ExampleEntity firstEntity = exampleEntityList.get(0);
            assertEquals(exampleDTO.getDescription(), firstEntity.getDescription());
            assertEquals(exampleDTO.getName(), firstEntity.getName());
        });
        producerTest.close();
    }
}
```
{% endcode %}

### Test d'intégration Producer

Pour vérifier que notre `ProducerService` à bien publié un objet `ExampleDTO` sur le topic `TOPIC_EXAMPLE_EXTERNE`. On crée un consumer dans qui va servir uniquement dans le cadre du test à écouter le topic `TOPIC_EXAMPLE_EXTERNE` afin de vérifier si un objet `ExampleDTO` à bien été envoyé dessus. On vérifie ensuite que l'objet reçu correspond bien à celui qu'on à envoyé précédemment.

{% code title="ProducerServiceIntegrationTest.java" %}
```java
package com.example.integrationtestspringkafka.service;

import com.example.integrationtestspringkafka.dto.ExampleDTO;
import com.example.integrationtestspringkafka.repository.ExampleRepository;
import org.apache.kafka.clients.consumer.Consumer;
import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.kafka.core.ConsumerFactory;
import org.springframework.kafka.core.DefaultKafkaConsumerFactory;
import org.springframework.kafka.support.serializer.JsonDeserializer;
import org.springframework.kafka.test.EmbeddedKafkaBroker;
import org.springframework.kafka.test.context.EmbeddedKafka;
import org.springframework.kafka.test.utils.KafkaTestUtils;
import org.springframework.test.annotation.DirtiesContext;
import org.springframework.test.context.junit.jupiter.SpringExtension;

import java.util.Map;

import static org.junit.jupiter.api.Assertions.assertEquals;

@ExtendWith(SpringExtension.class)
@SpringBootTest
@DirtiesContext
@EmbeddedKafka(topics = {"TOPIC_EXAMPLE", "TOPIC_EXAMPLE_EXTERNE"})
public class ProducerServiceIntegrationTest {
    private static final String TOPIC_EXAMPLE_EXTERNE = "TOPIC_EXAMPLE_EXTERNE";

    @Autowired
    private EmbeddedKafkaBroker embeddedKafkaBroker;

    @Autowired
    private ProducerService producerService;

    @Autowired
    private ExampleRepository exampleRepository;

    public ExampleDTO mockExampleDTO(String name, String description) {
        ExampleDTO exampleDTO = new ExampleDTO();
        exampleDTO.setDescription(description);
        exampleDTO.setName(name);
        return exampleDTO;
    }

    /**
     * We verify the output in the topic. With an simulated consumer.
     */
    @Test
    public void itShould_ProduceCorrectExampleDTO_to_TOPIC_EXAMPLE_EXTERNE() {
        // GIVEN
        ExampleDTO exampleDTO = mockExampleDTO("Un nom", "Une description");
        // simulation consumer
        Map<String, Object> consumerProps = KafkaTestUtils.consumerProps("group_consumer_test", "false", embeddedKafkaBroker);
        consumerProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        ConsumerFactory cf = new DefaultKafkaConsumerFactory<String, ExampleDTO>(consumerProps, new StringDeserializer(), new JsonDeserializer<>(ExampleDTO.class, false));
        Consumer<String, ExampleDTO> consumerServiceTest = cf.createConsumer();

        embeddedKafkaBroker.consumeFromAnEmbeddedTopic(consumerServiceTest, TOPIC_EXAMPLE_EXTERNE);
        // WHEN
        producerService.send(exampleDTO);
        // THEN
        ConsumerRecord<String, ExampleDTO> consumerRecordOfExampleDTO = KafkaTestUtils.getSingleRecord(consumerServiceTest, TOPIC_EXAMPLE_EXTERNE);
        ExampleDTO valueReceived = consumerRecordOfExampleDTO.value();

        assertEquals("Une description", valueReceived.getDescription());
        assertEquals("Un nom", valueReceived.getName());

        consumerServiceTest.close();
    }

}
```
{% endcode %}

## Resources

Github sources : 

* [github.com/Kevded/integration-test-spring-kafka-with-embedded-kafka-consumer-and-producer](https://github.com/Kevded/integration-test-spring-kafka-with-embedded-kafka-consumer-and-producer)

Exemple officiel spring-kafka : 

* [github.com/spring-projects/spring-kafka](https://github.com/spring-projects/spring-kafka)

Pour rédiger des tests d'intégration vous pouvez jeter un œil à :

* [github.com/authorjapps/zerocode](https://github.com/authorjapps/zerocode)
* [github.com/authorjapps/zerocode/tree/master/kafka-testing](https://github.com/authorjapps/zerocode/tree/master/kafka-testing)

