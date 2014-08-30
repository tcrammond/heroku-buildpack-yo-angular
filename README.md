Heroku buildpack: Yo Angular
============================

Parent Repository
-----------------

Refer to the parent repo's [readme](https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt/blob/master/README.md) **First**.

Additional Features
-------------------

- Installs bower apps from a `bower.json` file in the root directory
- Installs compass
- Deletes a few common files that are not need for production by Angular apps to reduce the slug size

### Current files being deleted to reduce slug size:

- .bowerrc
- .editorconfig
- .git/
- .gitattributes
- .gitignore
- .jshintrc
- .travis.yml
- .tmp/
- .sass-cache/
- Gruntfile.js
- app/
- bower_components/
- bower.json
- karma-e2e.conf.js
- karma.conf.js
- package.json
- test/

How to make your angular app work with this buildpack
-----------------------------------------------------

This buildpack uses a task in your `Gruntfile` to build the contents of your `app` folder into an optimized `dist` folder. This is the same thing the command `grunt` does in your `yo angular` app. All of this takes place on the heroku server, so all you have to do is push your app's source code to heroku with the standard `.gitignore` file . There is no need for you to commit the `dist` folder or any other pre-built files like a lot of the other online guides suggest. This buildpack installs all the `node_modules`, `bower_components` and uses `grunt` to build your app.

### 4 Step Guide

#### Step 1: Set up your `node.js` web server and add a `Procfile` command

I use `express`, `gzippo` and `morgan` with the `Procfile` command `web: node web.js`. But you can use anything. Here is a sample of my `web.js` file:

```js
'use strict';

var gzippo = require('gzippo');
var express = require('express');
var logger = require('morgan');
var nodeApp = express();

nodeApp.use(logger('dev'));
nodeApp.use(gzippo.staticGzip('' + __dirname + '/dist'));
nodeApp.listen(process.env.PORT || 5000);
```

#### Step 2: Add a heroku task to your Gruntfile

```js
grunt.registerTask('heroku', ['build']);
```

This is the task that will be run by heroku on the heroku server. I have just included the `build` task here (referring to the standard `build` task created by the `yo angular`) but you could add the `newer:jshint` and the `test` task too.

You can also set the environment variable `NODE_ENV` on your heroku server if you want the buildpack to call a `heroku:$NODE_ENV` task. In that case, here is how the task will look like in your `Gruntfile`:

```js
grunt.registerTask('heroku', function (target) {
  // use the target to do whatever, for example:
  grunt.task.run('build:' + target);
});
```

#### Step 3: Set this buildpack as your heroku app's buildpack

```
heroku config:set BUILDPACK_URL=https://github.com/nknj/heroku-buildpack-yo-angular.git
```

#### Step 4: Deploy

```
git push heroku master
```
