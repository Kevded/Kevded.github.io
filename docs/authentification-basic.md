---
description: >-
  "Sécuriser" simplement une application sur Firebase. Exemple empêcher l'accès
  à l'environnement de recette.
---

# Authentification HTTP Basic Header avec Firebase functions

Attention ne fonctionne pas en local avec "firebase serve" ou le shell firebase. Il faut obligatoirement déployer la fonction pour pouvoir tester.

{% code title="functions/index.js" %}
```javascript
'use strict';
const functions = require("firebase-functions");

const path = require("path");


const functions = require('firebase-functions');

// CORS Express middleware to enable CORS Requests.
const cors = require('cors')({origin: true});
exports.auth = functions.https.onRequest((req, res) => {
 
    // check for basic auth header
    if (!req.headers.authorization || req.headers.authorization.indexOf('Basic ') === -1) {
      // le navigateur détecte automatiquement l'header 
      // et ouvre une boite de dialogue avec les champs User/Password
      res.status(401).setHeader("WWW-Authenticate", "Basic");
      return res.end();
  }

  // verify auth credentials
  const base64Credentials = req.headers.authorization.split(' ')[1];
  const credentials = Buffer.from(base64Credentials, 'base64').toString('ascii');
  const [username, password] = credentials.split(':');
  if (username === "demo" && password === "demo") {
      console.log(username, "successfully logged in");
      // rediriger vers l'application
      // etant donnee qu'on est sur une single page application
      // une seule authentification est nécessaire pour charger 
      // l'application dans le navigateur
      return res.sendFile(path.join(__dirname, "public/index.html"));
  }
  console.log(username, "failed to logged in with password: ", password);
  res.status(401).setHeader("WWW-Authenticate", "Basic");
  return res.send("Unauthorized");
});

```
{% endcode %}

{% code title="functions/package.json" %}
```javascript


```
{% endcode %}

A placer dans le dossier functions. La partie hosting ne devra comporter que les assets\( Images, CSS\). Il ne devra pas contenir de fichier avec le nom "index.html".

{% code title="fucntions/public/index.html" %}
```markup
<!doctype html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <title>Application Sample</title>
 
  <meta name="viewport" content="width=device-width, initial-scale=1">
 
  <!-- GOOGLE FONT -->
  <link href="https://fonts.googleapis.com/css?family=Montserrat:400,600,700" rel="stylesheet" type="text/css">

<body>
  <noscript>Please enable JavaScript to continue using this application.</noscript>
  
  <script type="application/javascript">
  alert("Vous avez accéder à la page.");
  </script>
</body>

</html>
```
{% endcode %}



Inspiré de : [https://www.dotnetcurry.com/nodejs/1231/basic-authentication-using-nodejs](https://www.dotnetcurry.com/nodejs/1231/basic-authentication-using-nodejs)

