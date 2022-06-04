---
title: "Packing JS projects for AWS Lambda with Grunt"
description: "A sample Grunt configuration to pack JS projects for AWS Lambda."
date: 2020-05-02T20:40:46+02:00
categories:
- tech
tags:
- javascript
- lambda
cover:
    image: "posts/packing-js-projects-for-aws-lambda-with-grunt/assets/le-spui-a-la-haye-1868-johan-barthold-jongkind.jpg"
    alt: "Le Spui A La Haye (1868) - Johan Barthold Jongkind"
    relative: false
images: ["assets/le-spui-a-la-haye-1868-johan-barthold-jongkind.jpg"]
---

Grunt is a Javascript task runner. It automates repetitive tasks like
minification, compilation, unit testing, linting, etc. So it's also quite useful
for packing JS projects for AWS Lambda.

Install the CLI globally:

```bash
npm install -g grunt-cli
```

Add it to the `package.json`:

```bash
npm install grunt --save-dev
```

The `Gruntfile.js` or `Gruntfile.coffee` file is a valid JavaScript or
CoffeeScript file that belongs in the root directory of your project.

A Gruntfile is comprised of the following parts:

- The "wrapper" function
- Project and task configuration
- Loading Grunt plugins and tasks
- Custom tasks

A sample Gruntfile is as follows:

```js
module.exports = function (grunt) {
  // load all grunt plugins
  require("load-grunt-tasks")(grunt);

  // variables
  var buildPath = "./tmp/build/";
  var destFile = "./dest/index.zip"

  grunt.initConfig({
    copy: {
      build: {
        src: [
          "./src/**",
          "./index.js",
          "./package-lock.json",
          "./package.json",
          "./client_config.json",
        ],
        dest: buildPath,
      },
    },
    exec: {
      build: {
        cwd: buildPath,
        cmd: "npm ci --production",
      },
    },
    zip: {
      build: {
        cwd: buildPath,
        src: buildPath + "**",
        dest: destFile,
        compression: "DEFLATE",
      },
    },
  });

  // default task
  grunt.registerTask("default", ["build"]);

  // build task
  grunt.registerTask("build", ["copy", "exec", "zip"]);
};
```

`package.json` can then be configured for `npm build`:

```json
"scripts": {
  "build": "grunt"
}
```

Cheers.
