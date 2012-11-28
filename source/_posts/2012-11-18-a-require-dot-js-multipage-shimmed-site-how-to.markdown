---
layout: post
title: "A Require.js multi-page shimmed site: How-To"
date: 2012-11-18 13:05
comments: true
categories: [Require.js, R.js, Grunt]
---

I've been working with require.js a lot lately and found it really improves the way I manage my code. I had previous experience with require in the context of some of my Backbone posts but I'd never used it on a more traditional multi-page site before. The workflow to setup require on a multi-page site can be a little confusing at first so I'll try to walk you through the specifics.

<!--more-->

## Overview

*Note: This tutorial assumes you're already familiar with require.js and its configuration options. If not I recommend you [checkout the docs](http://requirejs.org/) before proceeding -- Rob*

## [Grab the Example Source](https://github.com/robdodson/requirejs-multipage-shim-tutorial)

Typically when you work with require.js you're building a single-page app. You usually have one main file which kicks off the app and pulls in dependencies as needed. When you're developing you leave these dependencies in separate files but when you move to production you need to combine them. In a production setting you want to cut down the number of http requests to the bare minimum. To assist you, require.js provides a build tool called r.js which minifies and concatenates all of your dependencies into one file.

When working with a multi-page site compiling everything into one file is probably not a good idea because you have no guarantee that the user will visit every page. Thus the ideal scenario is one in which each page has its own `main` file that contains page-specific instructions and then a separate (hopefully cached) file which contains all of the commonly used js libs. For example if you have a page called **About** and a page called **Contact** you would have `main-about.js` and `main-contact.js`. Let's say `main-about.js` and `main-contact.js` both require `'jquery'` and `'underscore'`. We wouldn't want to compile `jquery` and `underscore` into each `main` file because then we're creating unnecessary bloat. Instead we should create a `common.js` file which contains `jquery` and `underscore` and we'll make sure that this file loads before any of our `main` files. Take a look at this diagram to help it sink in:

**BEFORE OPTIMIZATION**
```
About
----------------
js/vendor/jquery.js
js/vendor/underscore.js
js/app/main-about.js

Contact
----------------
js/vendor/jquery.js
js/vendor/underscore.js
js/app/main-contact.js
```

**AFTER OPTIMIZATION**
```
common.js
----------------
js/vendor/jquery.js
js/vendor/underscore.js

About
----------------
js/common.js
js/app/main-about.js

Contact
----------------
js/common.js
js/app/main-contact.js
```

By compiling all of those libaries into `common.js` we're reducing the number of http requests per page. Also, after the first page has loaded, `common.js` should be cached available from the browser's cache. Now that you get the general concept let's take a look at an example.

## Examples!

[James Burke](https://twitter.com/jrburke), author of require.js, has done a great job putting together some example projects which show how to effectively structure your project so it can be run through the r.js optimization tool when you're ready to deploy. The one I typically refer to is the [example-multipage-shim.](https://github.com/requirejs/example-multipage-shim) There's also an [example-multipage](https://github.com/requirejs/example-multipage) but I tend to use the shim version because it seems like there's always a few non-AMD scripts that end up in a project.

I've put together my own boilerplate which you guys are free to grab [here.](https://github.com/robdodson/requirejs-multipage-shim-tutorial) The rest of this post will walk through this boilerplate.

## Learn you a boilerplate!

If you've worked with require.js on a single page site you're probably used to defining a script tag that looks like this:

```html
<!--This sets the baseUrl to the "scripts" directory, and
    loads a script that will have a module ID of 'main'-->
<script data-main="scripts/main.js" src="scripts/require.js"></script>
```

The `data-main` attribute is a convenience feature of require.js which sets require's [baseUrl property.](http://requirejs.org/docs/api.html#config-baseUrl) You usually put some configuration options in the `main` file as well, for instance, if you're loading a 3rd party library you might create a [path](http://requirejs.org/docs/api.html#config-paths) so you can easily reference it. Since our boilerplate has different `main` files for each page we're going to put that configuration data into our `common.js` file instead.

*Hold on, I thought you said common.js was where we were going to compile all of our libraries?*

Indeed you are correct astute reader. But since `common.js` is going to be loaded before any other required modules, why not put our configuration options in it as well? Here's what the `common.js` file from our boilerplate looks like:

``` js common.js
//The build will inline common dependencies into this file.

requirejs.config({
  baseUrl: './js',
  paths: {
    'jquery': 'vendor/require-jquery',
    'bootstrap': 'vendor/bootstrap'
  },
  shim: {
    'bootstrap': [ 'jquery' ]
  }
});
```

To smooth over possible weird module shimming situations I tend to use the combined jQuery-require.js bundle available [here.](http://requirejs.org/docs/jquery.html) Then in my paths option I map `jquery` to this file. The file will already be loaded on the page so it doesn't actually request anything new, it just makes sure that `$` is available in my modules.

You'll also note that we're using [Twitter Bootstrap](http://twitter.github.com/bootstrap/) which is not AMD compliant out of the box so we have to shim it. In this case it depends on `jquery` so we list that as its only dependency and we're good.

Aside from `common.js` you should probably take a look in the `app/models` directory. I've created two models which extend a base model in order to illustrate that the `common.js` file isn't only for 3rd party code, it's really for any code that's going to be used over and over on your site. We'll toss the basicModel into common.js during the build process.

## Just hit build already...

OK ok. I can tell you're ready to see some fireworks. Before we hit build I want to point out the `options.js` file. This is where we tell r.js which modules to compile and what each module should include or exclude.

``` js options.js
module.exports = {
  appDir: 'www',
  baseUrl: 'js/',
  mainConfigFile: 'www/js/common.js',
  dir: 'www-release',
  modules: [
    {
      name: 'common',
      include: [
        'app/models/basicModel',
        'bootstrap'
      ],
      exclude: ['jquery']
    },

    {
      name: 'app/main-about',
      exclude: ['common', 'jquery']
    },
    
    {
      name: 'app/main-contact',
      exclude: ['common', 'jquery']
    }
  ]
};
```

You'll note that first we put together our common module, then we tell the subsequent modules to exclude it. When you tell r.js to exclude a module it will find all of the nested dependencies in that module and exclude those as well. This is why we don't need to tell about and contact to exclude bootstrap. It sees that bootstrap is already in common so it knows to exclude it.

If you haven't already now is a good time to do an `npm install`. That should pull down all of the grunt dependencies. Speaking of grunt, you'll need to install that as well if you've never used it `npm install -g grunt`.

OK. Ready to rock. Just type `grunt` to build! When you're finished you should have a new folder called `www-release`. Take a look at the `build.txt` file to see where everything ended up.

```
css/bootstrap-responsive.css
----------------
css/bootstrap-responsive.css

css/bootstrap.css
----------------
css/bootstrap.css

css/style.css
----------------
css/style.css

js/common.js
----------------
js/common.js
js/app/models/basicModel.js
js/vendor/bootstrap.js

js/app/main-about.js
----------------
js/app/models/aboutModel.js
js/app/main-about.js

js/app/main-contact.js
----------------
js/app/models/contactModel.js
js/app/main-contact.js
```

If you take a look at the `common.js` file in `www-release` it should look like a big blob of minified code. That's what we want to see. In this case it contains our basicModel, twitter bootstrap and our original configuration code. If you refer to about.html you can see how we control the load order:

``` html
<script src="./js/vendor/require-jquery.js"></script>
<script type="text/javascript">
// Load common code that includes config,
// then load the app logic for this page.
require(['./js/common'], function (common) {
  // ./js/common.js sets the baseUrl to be ./js/
  // You can ask for 'app/main-about' here instead
  // of './js/app/main-about'
  require(['app/main-about']);
});
</script>
```

First we bring in our combined require.js/jquery script. Then we load `common.js` and only *after* `common.js` is loaded do we load the page specific code in `main-about`. If you stick to this structure you should be able to easily layer your code so it's easy to manage throughout your site.

## [Grab the Example Source](https://github.com/robdodson/requirejs-multipage-shim-tutorial)

Hope you learned something new today. Feedback and suggestions always welcome in the comments below. Thanks!

You should follow me on Twitter [here.](http://twitter.com/rob_dodson)