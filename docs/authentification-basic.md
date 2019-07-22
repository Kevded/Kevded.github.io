---
description: >-
  "Sécuriser" simplement une application sur Firebase. Exemple empêcher l'accès
  à l'environnement de recette.
---

# Authentification HTTP Basic Header avec Firebase functions

{% code-tabs %}
{% code-tabs-item title="index.js" %}
```javascript
'use strict';

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
      res.status(200).setHeader("Content-Type", "text/plain");
      // rediriger vers l'application
      // etant donnee qu'on est sur une single page application
      // une seule authentification est nécessaire pour charger 
      // l'application dans le navigateur
      return res.send("ok");
  }
  console.log(username, "failed to logged in with password: ", password);
  res.status(401).setHeader("WWW-Authenticate", "Basic");
  return res.send("Unauthorized");
});

```
{% endcode-tabs-item %}
{% endcode-tabs %}

