---
layout: post
title: "Debug LESS with Chrome Developer Tools"
date: 2012-12-28 08:32
comments: true
categories: [LESS, Chrome, CodeKit, Less-Middleware, LESS.js]
---

*Update: Chrome 27+ now uses v3 sourcemaps which LESS 1.3.3/1.4-beta does not support. That means you won't be able to inspect the actual styles in the dev tools, however, if you click on the link to the CSS file in the inspector, the sourcemap will show you which LESS file is generating the output. This can still be really handy in a project with a ton of LESS files. Please see [Paul Irish's comments below for more details.](http://robdodson.me/blog/2012/12/28/debug-less-with-chrome-developer-tools/#comment-919144010)*

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
This is known as a **source map** and it basically tells a debugging tool how to find its way from the generated output back to the correct source file. Thankfully the current version of Chrome (26, as of this writing) supports source maps for SASS and the current version of LESS (1.3.3, as of this writing) outputs source maps that *look* like SASS source maps to take advantage of this.

*Important: If you use less.js to compile your LESS in the browser the techniques we'll be covering will not work for you. Unfortunately less.js generates all its output in a big style block at the top of the page and that seems to confuse the dev tools. I wanted to point that out before you spend too much time setting things up.*

## Setting Up Chrome

Before we begin make sure you're running a version of Chrome that's 24 or above. **At present Chrome Canary 28 does not work with this technique so you need to use regular Chrome 24 - 26. I'm pinging the Chrome dev team to see what's up.**.

In the address bar type `chrome://flags/` and hit enter. You should be transported to a magical place.

{% img center https://s3.amazonaws.com/robdodson/images/chrome-flags.png 'The Chrome experiments section' %}

Here we'll search for "Enable Developer Tools experiments". When you find it click "Enable". Then **restart Chrome**. Don't just close the window, actually quit the program and restart it. Once it's reopened fire up the developer tools and click the gear in the bottom right.

{% img center https://s3.amazonaws.com/robdodson/images/chrome-options.png 'Chrome Developer tools options' %}

In the left hand sidebar click General. Scroll down to where it says Sources and click "Enable source maps". Again in the sidebar click Experiments, scroll down and enable "Support for Sass".

Now if you're just working with SASS then all you have to do is make sure your SASS files generate the proper source maps and you're done. [Here's a great article to walk you through the last couple of steps.](http://bricss.net/post/33788072565/using-sass-source-maps-in-webkit-inspector) But if you're like me and your codebase is in LESS there is more work to be done. Onward!

## Processors

There are a quite a few ways to convert your LESS into properly source mapped CSS code. You can use the lessc command line tool, a GUI such as [CodeKit](http://incident57.com/codekit/) or have the server do it with something like [less-middleware](https://github.com/emberfeather/less.js-middleware) for [connect](http://www.senchalabs.org/connect/)/[express.](http://expressjs.com/) As I mentioned previously, you can also compile less on the client-side using less.js but unfortunately our debugging technique does not seem to work with that approach so you'll need to use an alternative.

One quick warning, which I'll emphasize again later, **make sure you turn off all minification** otherwise it will break our source maps!

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

Your generated LESS should now be inspectable in Chrome. Yay! Make sure you're not running any kind of minification step further along in your build or it will break the source maps.

### CodeKit

In CodeKit use the **Debug Info in CSS** dropdown and set it to **Media Query at Top of CSS File**. Make sure you set **Output Style** to **Regular** otherwise the minification will break our source maps.

After that you should be good to go :)

### less-middleware

You might need to update your version of the middleware to whatever's the latest, which looks like 0.1.9 as of this writing.

In your app you'll need to add the `dumpLineNumbers` options to the middleware's config.

``` js app.js
app.use(lessMiddleware({
  src: __dirname + 'path/to/src',
  dest: __dirname + 'path/to/dest',
  dumpLineNumbers: 'mediaquery'
}));
```

After that you should be all set. Make sure any minification is turned off or it will screw up the source maps.

## Warnings

The above techniques work for regular Chrome 24 - 26 but **appear to be broken in Chrome Canary 28**.

**Minification of any kind seems to screw up the source maps** so make sure your code is not minified or YUI compressed. Remember this is for debugging your code so it's ok if it's not minified.

One more thing, make sure before you send your code into production that you remember to **turn off debugging** otherwise you'll be needlessly bloating your CSS files by quite a lot.

## Conclusion

Personally I've found this trick *extremely* useful when working with large LESS codebases. I've seen some chatter that Stylus might also support this trick so if you have first-hand experience debugging Stylus with Chrome please drop me a comment. Otherwise I might do a follow up showing how to achieve similar results in that language.

Any questions or comments hit me up in the discussion area below.

-- Rob

You should follow me on Twitter [here.](http://twitter.com/rob_dodson)


