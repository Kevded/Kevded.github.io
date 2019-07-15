---
description: NodeJS App Web and Worker process type
---

# Deploy monorepo in single app on Heroku

Requirements: 

assume that your have a monorepo folder tree like this:

```text
root(monorepo)
--webapp
----app.js
----package.json
--workerapp
----app.js
----package.json 
```

1. In your monorepo root create .buildpacks file 

   ```text
   webapp=https://github.com/heroku/heroku-buildpack-nodejs.git
   workerapp=https://github.com/heroku/heroku-buildpack-nodejs.git
   ```

2. Add a Procfile

   In root folder. 

   ```
   web: cd webapp && npm start
   worker: cd workerapp && npm start
   ```

3. Create an app &lt;my-app&gt; on heroku and set up git in your monorepo folder
4. Add Buildpack config in your &lt;my-app&gt; settings

   [https://github.com/negativetwelve/heroku-buildpack-subdir](https://github.com/negativetwelve/heroku-buildpack-subdir)

5. Git push heroku master
6. Scale your web et worker

   ```text
   heroku ps:scale web=1 worker=1
   ```



Ask me : [https://twitter.com/deddysaintval](https://twitter.com/deddysaintval)

