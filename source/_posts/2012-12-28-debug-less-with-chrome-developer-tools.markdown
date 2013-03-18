---
layout: post
title: "Debug LESS with Chrome Developer Tools"
date: 2012-12-28 08:32
comments: true
categories: [LESS, Chrome, CodeKit, Less-Middleware, LESS.js]
---

If you've spent much time with preprocessors like LESS, SASS/SCSS or Stylus you've probably discovered their one rather crippling flaw: debugging. With thousands of lines of LESS code suddenly turning into even more thousands of lines of CSS it can become nearly impossible to tell where a particular style comes from. Inspecting CSS used to be the domain of the Chrome Developer Tools and Firebug but now that our CSS is machine generated there's no longer a link between the style at line 2137 and the LESS file that generated it. Thankfully the Chrome team is addressing this problem but their current focus is on SASS. Today I'll teach you how to rework your LESS processor so it plays nice with Chrome and reunites you with your old friend, the CSS inspector.

<!--more-->

Just to whet your appetite here's a teaser shot of what we're going to accomplish.

{% img center https://s3.amazonaws.com/robdodson/images/less-preview.png 'A preview of the Inspector showing LESS debugging' %}

You'll notice over on the right instead of your typical `style.css: 7` it says `modules.less: 7`. That's right, the developer tools are looking at the generated CSS and source mapping it back to the LESS files.

{% img center https://s3.amazonaws.com/robdodson/images/less-preview2.png 'The inspector revealing the actual LESS file' %}

Clicking on the line number will actual dive into the LESS file where we can see the nesting, variables and mixins.

"Awesome!" you say, but how do we do it?

Well SASS has a debugging feature which will output media-queries above each style. It looks like this:

``` css
@media -sass-debug-info{filename{font-family:file\:\/\/\/Users\/Rob\/Desktop\/less-debug\/less\/base\.less}line{font-family:\000035}}
h1 {
  color: #999999;
}
```
This is known as a **source map** and it basically tells a debugging tool how to find it's way from the generated output back to the correct source file. The latest version of Chrome Canary has specific support for SASS style source maps, so [Felix Gnass cleverly reasoned that if LESS could work like SASS in this respect then the developer tools would pick up on that as well.](https://github.com/cloudhead/less.js/pull/1038) LESS already has a similar source map feature but the media-queries didn't *quite* use the same syntax. In LESS 1.3.3 this should all be fixed so we can now debug our LESS files using the Chrome Dev Tools!

Let's go ahead and setup Chrome Canary which is required because these are still experimental features.

*Important: If you use less.js to compile your LESS in the browser the techniques we'll be covering will not work for you. Unfortunately less.js generates all its output in a big style block at the top of the page and that seems to confuse the dev tools. I wanted to point that out before you spend too much time setting things up.*

## Chrome Canary

You can grab the latest version of [Chrome Canary here.](https://www.google.com/intl/en/chrome/browser/canary.html) At the time of this writing (Jan. 09, 2013) the standard Chrome browser does not yet have this feature so Canary is a must.

Once you've downloaded Canary type `chrome://flags/` into the address bar and hit enter. You should be transported to a magical place.

{% img center https://s3.amazonaws.com/robdodson/images/chrome-flags.png 'The Chrome Canary experiments section' %}

Here we'll search for "Enable Developer Tools experiments". When you find it click "Enable". Then **restart Canary**. Once it's open again fire up the developer tools and click the gear in the bottom right.

{% img center https://s3.amazonaws.com/robdodson/images/chrome-options.png 'Chrome Developer tools options' %}

In the left hand sidebar click General. Scroll down to where it says Sources and click "Enable source maps". Again in the sidebar click Experiments, scroll down and enable "Support for Sass".

Now if you're just working with SASS then all you have to do is make sure your SASS files generate the proper source maps and you're done. [Here's a great article to walk you through the last couple of steps.](http://bricss.net/post/33788072565/using-sass-source-maps-in-webkit-inspector) But if you're like me and your codebase is in LESS there is more work to be done. Onward!

## Processors

There are a quite a few ways to convert your LESS into properly source mapped CSS code. You can use the lessc command line tool, a GUI such as [CodeKit](http://incident57.com/codekit/) or have the server do it with something like [less-middleware](https://github.com/emberfeather/less.js-middleware) for [connect](http://www.senchalabs.org/connect/)/[express.](http://expressjs.com/) As I mentioned previously, you can also compile less on the client-side using less.js but unfortunately our debugging technique does not seem to work with that approach so you'll need to use an alternative.

Depending on your processor and the version of LESS it uses you might have to locate the `tree.js` file inside its copy of LESS and change some of the debugging output. I know that the current version of CodeKit (1.4.1) still uses LESS 1.3.1 so we'll detail how to patch it.

### lessc

If you've installed LESS using npm check that you've got the latest version.

``` bash
$ lessc --version
lessc 1.3.3 (LESS Compiler) [JavaScript]
```
If your version is not 1.3.3 or greater you should run `npm install -g less` to update to the latest version.

To compile your less with baked in source maps pass the `--line-numbers` flag a value of `mediaquery`. For example:

``` bash
lessc --line-numbers=mediaquery theme.less theme.css
```

Your generated LESS should now be inspectable in Chrome Canary. Yay!

### CodeKit

We'll have to patch CodeKit 1.4.1 because it's still using an oudated version of LESS. Navigate to your Applications folder and right-click on CodeKit.app. Choose "Show Package Contents." Navigate to `Contents/Resources/engines/less/lib/less/` and open `tree.js`.

Inside of `tree.js` we're looking for a block of code like this:

``` js tree.js
tree.debugInfo.asMediaQuery = function(ctx) {
    return '@media -sass-debug-info{filename{font-family:"' + ctx.debugInfo.fileName + '";}line{font-family:"' + ctx.debugInfo.lineNumber + '";}}\n';
};
```
we're just going to replace that function with this one:

``` js tree.js
tree.debugInfo.asMediaQuery = function(ctx) {
    return '@media -sass-debug-info{filename{font-family:' +
        ('file://' + ctx.debugInfo.fileName).replace(/[\/:.]/g, '\\$&') +
        '}line{font-family:\\00003' + ctx.debugInfo.lineNumber + '}}\n';
};
```
and that's it really!

Now when you go back to CodeKit use the "Debug Info in CSS" dropdown and set it to "Media Query at Top of CSS File".

After that you should be good to go :)

### less-middleware

If you're using the [less-middleware](https://github.com/emberfeather/less.js-middleware) extension for Express you'll want to check its `package.json` and make sure it's on version 0.1.9. You should notice in its dependencies that it requires LESS 1.3.3.

``` js
"dependencies": {
    "less": ">= 1.3.3",
    "mkdirp": ">= 0.3.1"
  },
```
You might need to update your version of the middleware and run `npm install` again to update everything to the very latest.

In your app you'll need to add the `dumpLineNumbers` options to the middleware's config.

``` js app.js
app.use(lessMiddleware({
  src: __dirname + 'path/to/src',
  dest: __dirname + 'path/to/dest',
  dumpLineNumbers: 'mediaquery'
}));
```

After that you should be all set.

## Warnings

Be careful about hacking your own `tree.js` file if you can't update LESS for whatever reason. For instance if you're using CodeKit a future update might wipe out your patch but not necessarily bring LESS to the current version so you may have to re-apply it.

One more thing, make sure before you send your code into production that you remember to **turn off debugging** otherwise you'll be needlessly bloating your CSS files by quite a lot.

## Conclusion

Personally I've found this trick *extremely* useful when working with large LESS codebases. I've seen some chatter that Stylus might also support this trick so if you have first-hand experience debugging Stylus with Chrome please drop me a comment. Otherwise I might do a follow up showing how to achieve similar results in that language.

Any questions or comments hit me up in the discussion area below.

-- Rob

You should follow me on Twitter [here.](http://twitter.com/rob_dodson)


