---
description: Exemple de solution avec pipedream.com
---

# Exposer un document Google Sheets privé via API REST

Date de publication : 08/03/2021

Par défaut l’API Google Sheets permet l'accès à un fichier Google Sheets uniquement via authentification OAuth 2.0 ou via API key.

La solution via OAuth 2.0 nécessite une connexion avec les crédentials Google et une autorisation. Tandis que la solution via API key permet d'exposer uniquement les documents Google Sheets partagé publiquement au préalable. 

### Solution avec pipedream

Avec [pipedream.com ](https://pipedream.com)nous allons exposer notre document Google Sheets privé via une API REST.

Retourner le nombre total de lignes remplis et les valeurs. Sous la forme `{"rows" : 2 , "values" : {} }`

Remarque : l'API n'est pas sécurisé vous êtes libre d'ajouter une étape au workflow pour ajouter de l'authentification.

Prérequis : 

* Connecter votre compte \(ou un compte de test\) Google et autoriser l'accès
* Spécifier le l'ID du Google Sheets
* Spécifier le nom du Sheet
* Déployer le workflow

Une fois déployé une URL est généré vous pouvez copier/coller l'URL dans votre navigateur pour afficher les résultats.

[Lien du workflow pipedream](https://pipedream.com/@kevded/get-count-filled-rows-and-expose-values-google-sheets-public-p_n1CKnZJ)



