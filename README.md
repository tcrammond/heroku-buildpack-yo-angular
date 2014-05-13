# Heroku buildpack: Yo Angular

## Parent Repository
Refer to the parent repo's [readme](https://github.com/mbuchetics/heroku-buildpack-nodejs-grunt/blob/2168fa2da5e7c8adf4ef60922afc2b0199f404de/README.md) for more information. This fork was made at commit 2168fa2. The readme link refers to the `README.md` file at that same commit.

## Key Differences
- The parent repository install bower apps from a `bower.json` file in the root directory, this does

- This repository deletes a few common files that are not need for production to reduce the slug size
This reduces the slug size by almost 70% in most of my repositories. I don't typically use any of these files I am getting rid of but if you do, please fork and edit this for yourself.

### Current files being deleted:
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
- node_modules/*grunt*
- node_modules/*karma*
- node_modules/jshint*
- node_modules/bower

## How this works
This buildpack uses a task in your `Gruntfile` to build your `app` into production ready `dist` the same way the command `grunt` does. It does this on the heroku server, so all you have to do is push your angular app to heroku. There is no need for you to commit the `dist` folder or any pre-compiled files like a lot of the other online guides suggest.

### How to make it work:

#### Step 1: `devDependecies` to `dependencies` in `packages.json`
Move all the grunt packages in `devDependencies` that are required for your build task to `dependencies`. The build takes place on the heroku server, so these packages are needed. At the end of the compilation, most of these dependencies are deleted to reduce your slug size.

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

#### Step 3: Set this buildpack as your heroku app's buildpack
```
heroku config:set BUILDPACK_URL=https://github.com/nknj/heroku-buildpack-yo-angular.git
```

#### Step 4: Deploy
```
git push heroku master
```

Pull requests welcome :)
