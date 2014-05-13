# Heroku buildpack: Yo Angular

## Parent Repository
Refer to the parent repo's [readme](https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt/blob/2168fa2da5e7c8adf4ef60922afc2b0199f404de/README.md) **first**. This fork was made at commit 2168fa2. The readme link refers to the `README.md` file at that same commit.

## Features
- Installs bower apps from a `bower.json` file in the root directory

- Deletes a few common files that are not need for production by Angular apps to reduce the slug size. This reduces the slug size by almost 70% in most of my repositories.  

**WARNING:** As far as I know, none of these files and folders are or should be used in a production environment. They are only used for tests or builds. If you are using any of the following files for production, this buildpack is likely to give you problems.

### Current files being deleted to reduce slug size:
- .bowerrc
- .editorconfig
- .git/
- .gitattributes
- .gitignore
- .jshintrc
- .travis.yml
- .tmp/
- Gruntfile.js
- app/
- bower.json
- karma-e2e.conf.js
- karma.conf.js
- package.json
- test/
- node_modules/\*grunt\*
- node_modules/\*karma\*
- node_modules/jshint\*
- node_modules/bower

## How to make your angular app work with this buildpack
This buildpack uses a task in your `Gruntfile` to build the contents of your `app` folder into an optimized `dist` folder. This is the same thing the command `grunt` does in your `yo angular` app. All of this takes place on the heroku server, so all you have to do is push your app's source code to heroku with the standard `.gitignore` file . There is no need for you to commit the `dist` folder or any other pre-built files like a lot of the other online guides suggest. This buildpack installs all the `node_modules`, `bower_components` and uses `grunt` to build your app.

### 5 Step Guide

#### Step 1: `devDependecies` to `dependencies` in `packages.json`
Move all the grunt packages in `devDependencies` that are required for your build task to `dependencies`. The build takes place on the heroku server, so these packages are needed on the server. At the end of the compilation, most of these dependencies are deleted to reduce your slug size.

#### Step 2: Set up your `node.js` web server and add a `Procfile` command
I use `express` and `gzippo` with the `Procfile` command `web: node web.js`. But you can use anything. Here is a sample of my `web.js` file:

```js
'use strict';

var gzippo = require('gzippo');
var express = require('express');
var nodeApp = express();

nodeApp.use(express.logger('dev'));
nodeApp.use(gzippo.staticGzip('' + __dirname + '/dist'));
nodeApp.listen(process.env.PORT || 5000);
```

#### Step 3: Add a heroku task to your Gruntfile
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

#### Step 4: Set this buildpack as your heroku app's buildpack
```
heroku config:set BUILDPACK_URL=https://github.com/nknj/heroku-buildpack-yo-angular.git
```

#### Step 5: Deploy
```
git push heroku master
```

Pull requests welcome :)
