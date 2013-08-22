---
layout: post
title: "Exploring HTML Imports"
date: 2013-08-20 13:07
comments: true
categories: [HTML5, HTML Imports, Polymer, Template, Web Components]
---

[Web Components](http://robdodson.me/blog/2013/03/17/why-web-components/) have come a long way in the past few months and one of the technologies that I'm most interested in is [HTML Imports](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/imports/index.html) (or "imports", for short). Imports allow you to load additional documents into your page without having to write a bunch of ajax. This is great for [Custom Elements](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html) where you might want to import a suite of new tags. I spent the day playing with imports and thought it would be useful to write up my progress.

<!--more-->

## The Lowdown

Imports are a new type of `link` tag which should be familiar to you since that's also how we load our stylesheets.

```html
<link rel="stylesheet" href="/path/to/styles.css">
```

For an import we just replace the `rel` with one of type `import`.

```html
<link rel="import" href="/path/to/some/import.html">
```

At the moment imports do not block like script tags, however, that may change in the future to help with [Custom Elements resolution.](http://lists.w3.org/Archives/Public/public-webapps/2013JulSep/0287.html)

## Support

Native imports are only available in Chrome Canary v31 and Chrome Dev v30. Thankfully [Polymer](http://www.polymer-project.org/) [offers a polyfill](http://www.polymer-project.org/platform/html-imports.html) if you want to try them out in other modern / "evergreen" browsers.

To use HTML Imports make sure you've enabled the following
in Chrome's `about:flags`.

√ Experimental Web Platform features<br>
√ Experimental JavaScript<br>
√ HTML Imports<br>

*I'm not 100% certain if these are all necessary but they don't hurt ;)*

## Codez!

I've created a sketchbook for this post and future Web Components related stuff. [You can grab the sketchbook on GitHub.](https://github.com/robdodson/webcomponents-sketchbook) For each of the examples that I cover I'll link to the sketch so you can quickly try things out.

## A Basic Example

### Sketch 0: [Basic](https://github.com/robdodson/webcomponents-sketchbook/tree/master/html-imports/0-basic)

OK so what's a very basic import look like?

``` html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Basic HTML Imports</title>
    <!-- Pull in our blog post example -->
    <link rel="import" href="/imports/blog-post.html">
  </head>
  <body>
    <p>Hello World!</p>
  </body>
</html>
```
In its simplest form the import works just like calling a stylesheet. We have to make sure there's a document to import so let's create a fake blog post.

``` html imports/blog-post.html
<div id="blog-post">
  <h1>Awesome header</h1>
  <p>Here is some really interesting paragraph content.</p>
</div>
```

To test you'll need to host your `index.html` and `imports/` folder on a local server. I recommend [serve](https://github.com/visionmedia/serve) if you don't already have one installed.

Once you have that setup visit your index page. If you take a look at the console you can see the request returning.

{% img center http://robdodson.s3.amazonaws.com/images/imports-console.jpg 'Our first HTML import!' %}

Well that's cool, but now what?

Let's grab the content of the import using some JavaScript and append it to the body.

``` html index.html
<body>
  <p>Hello World!</p>

  <script>
    var link = document.querySelector('link[rel=import]');
    var content = link.import.querySelector('#blog-post');
    document.body.appendChild(content.cloneNode(true));
  </script>
</body>
```
First we query the `link` tag which loaded our import. Then we extract our `#blog-post` element and store it in a variable called `content`. You'll notice that we don't have to write any event handler code to wait till the import has loaded, we can just assume the content is there and start working with it. Finally we add the new content to our `body`.

If you're following along you should end up with something like this:

{% img center http://robdodson.s3.amazonaws.com/images/imports-screen1.jpg 'Basic Import Example' %}

Exciting, I know ;) But it demonstrates a no frills approach to loading content that doesn't require ajax and writing our own event handlers. Let's keep digging to see what else we find...

## A Basic Example with Polymer

### Sketch 1: [Basic-Polymer](https://github.com/robdodson/webcomponents-sketchbook/tree/master/html-imports/1-basic-polymer)

If you want to try out the snippets above in a browser other than Chrome Canary you'll need to use Google's [Polymer Project](http://www.polymer-project.org/). Polymer is a collection of polyfills and additional sugars which seeks to enable the use of Web Components in all modern browsers. The hope is that devolopers will use Polymer to inform the W3C on which direction to take with Web Components; so rather than wait for a stinky spec we can guide the implementation process.

Polymer attempts to keep parity with the the evolving specifications but obviously there are some places where the API must differ because of the limitations of current browsers. In the case of HTML Imports, Polymer waits for the `DOMContentLoaded` event before triggering the actual import process. This means we need to listen for the `HTMLImportsLoaded` event on either `window` or `document` to know when it is finished.

``` html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Basic HTML Imports</title>
    <link rel="import" href="/imports/blog-content.html">
  </head>
  <body>
    <p>Hello World!</p>
    
    <!-- Include polymer.js -->
    <script src="/bower_components/polymer/polymer.min.js"></script>

    <!-- Listen for the HTMLImportsLoaded event -->
    <script>
      window.addEventListener('HTMLImportsLoaded', function() {
        var link = document.querySelector('link[rel=import]');
        var content = link.import.querySelector('#blog-post');
        document.body.appendChild(content.cloneNode(true));
      });
    </script>
  </body>
</html>
```
Using the above we should get the same results as before.

{% img center http://robdodson.s3.amazonaws.com/images/imports-screen1.jpg 'Basic Import Example with Polymer' %}

You might notice that I used `polymer.min.js` instead of only including the [HTML Imports polyfill](http://www.polymer-project.org/platform/html-imports.html). Polymer is structured so you can take any of the polyfills &agrave; la carte but I find it's easier to just include all of Polymer when I'm experimenting, rather than worry if I have each individual polyfill loaded. That's just personal preference (aka I'm lazy).

## Using CSS in our Imports

### Sketch 2: [CSS](https://github.com/robdodson/webcomponents-sketchbook/tree/master/html-imports/2-css)

Let's put Polymer to the side for a bit and go back to our original, native example.

One of the first things I wanted to try was importing documents using the new CSS [`scoped`](http://www.w3.org/TR/html51/document-metadata.html#attr-style-scoped) attribute. The `scoped` attribute allows you to include `<style>` tags inside of an element which **only affect that element and not the entire document**. Let's update our index file a bit to demonstrate:

``` html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Imports with CSS</title>
    <link rel="import" href="/imports/blog-post.html">
  </head>
  <body>
    <!-- Add a header so we can show off scoped -->
    <h1>Boring header</h1>
    <p>Hello World!</p>

    <script>
      var link = document.querySelector('link[rel=import]');
      var content = link.import.querySelector('#blog-post');
      document.body.appendChild(content.cloneNode(true));
    </script>
  </body>
</html>
```
The only change I've made is to add an `h1` at line 10 so we can better illustrate how `scoped` works. Now let's update our fake blog post.

``` html imports/blog-post.html
<div id="blog-post">
  <style scoped>
    h1 {
      background: lightgreen;
      color: green;
    }

    p {
      font-size: 16px;
      font-family: Helvetica, Arial, sans-serif;
      color: green;
      font-weight: bold;
    }
  </style>

  <h1>Awesome header</h1>
  <p>
    Here is some really interesting paragraph content.
    It comes with its own stylesheet!
  </p>
</div>
```
When we run this in Canary we should get the following:

{% img center http://robdodson.s3.amazonaws.com/images/imports-screen2.jpg 'Imports with Scoped Styles' %}

It's still not much to look at but the gears in my imagination are starting to turn now. We just imported a document, which has its own styles that are scoped specifically to it, and we stamped its contents onto our page. We're starting to get into Web Components territory and that's pretty exciting. Let's see what else we can do...

## Using Scripts in our Imports
### Sketch 3: [Script](https://github.com/robdodson/webcomponents-sketchbook/tree/master/html-imports/3-script)

Next let's look at using `<script>` tags inside of our import. We'll start by removing the `<script>` block from our `index.html`.

```html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Basic HTML Imports</title>
    <link rel="import" href="/imports/blog-post.html">
  </head>
  <body>
    <h1>Boring header</h1>
    <p>Hello World!</p>
  </body>
</html>
```

Then we'll transfer that script block over to our blog post.

```html imports/blog-post.html
<div id="blog-post">
  <style scoped>
    h1 {
      background: lightgreen;
      color: green;
    }

    p {
      font-size: 16px;
      font-family: Helvetica, Arial, sans-serif;
      color: green;
      font-weight: bold;
    }
  </style>

  <h1>Awesome header</h1>
  <p>
    Here is some really interesting paragraph content.
    It comes with its own stylesheet!
  </p>
</div>

<script>
  // thisDoc refers to the "importee", which is blog-post.html
  var thisDoc = document.currentScript.ownerDocument;

  // thatDoc refers to the "importer", which is index.html
  var thatDoc = document;

  // grab the contents of the #blog-post from this document
  // and append it to the importing document.
  var content = thisDoc.querySelector('#blog-post');
  thatDoc.body.appendChild(content.cloneNode(true));
</script>
```
If we run this we should get the exact same outcome as before.

{% img center http://robdodson.s3.amazonaws.com/images/imports-screen2.jpg 'Imports with Scoped Styles' %}

An important thing to take notice of is the relationship between `thisDoc` and `thatDoc`. `thisDoc` refers to the `blog-post.html` document while `thatDoc` refers to our index.html file. It's useful to distinguish between the two so we can `querySelector` for `#blog-post` and not worry that we may have grabbed something out of the importing document. *Thanks to [Dominic Cooney](https://twitter.com/coonsta) for the heads up on this.*

You'll also notice that since the import has access to our `document` object it is able to add itself to the page. In practice you probably wouldn't want imports adding themselves wherever but the important takeaway is that **anything imported can access the `document`**. This means an import could register itself as a Custom Element using our `document` object and we wouldn't need to write any additional code. We're almost to that point so let's keep going...

## Using Templates in our Imports
### Sketch 4: [Template](https://github.com/robdodson/webcomponents-sketchbook/tree/master/html-imports/4-template)

I'm getting a little tired of our fake "blog post" so let's switch over to something more practical. We'll use [Chart.js](http://www.chartjs.org/) to create a very simple pie diagram and we'll use the new `<template>` tag to hold the contents of our import. If you haven't heard of the template tag before [checkout this introduction](http://robdodson.me/blog/2013/03/16/html5-template-tag-introduction/).

To start, I've updated the `index.html` so it imports a new `chart.html` file.

```html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Imports with Templates</title>
    <!-- Make sure to import chart.html -->
    <link rel="import" href="/imports/chart.html">
  </head>
  <body>
    <h1>Quarterly Pokemon Sales</h1>
  </body>
</html>
```

And here's what `chart.html` actually looks like.

```html imports/chart.html
<template id="chart-pie">
  <canvas id="myChart" width="200" height="200"></canvas>
  <script>
    var data = [
      {
        value: 30,
        color:"#F38630"
      },
      {
        value : 50,
        color : "#E0E4CC"
      },
      {
        value : 100,
        color : "#69D2E7"
      }
    ];

    // Get the context of the canvas element we want to select
    // It's ok to use document here because this script block won't
    // activate till it's added to the page.
    var ctx = document.getElementById("myChart").getContext("2d");
    var myNewChart = new Chart(ctx).Pie(data);
  </script>
</template>

<script src="/lib/chart.min.js"></script>
<script>
  // thisDoc refers to the "importee", which is chart.html
  var thisDoc = document.currentScript.ownerDocument;

  // thatDoc refers to the "importer", which is index.html
  var thatDoc = document;

  // grab the contents of #chart-pie from this document
  // and append it to the importing document.
  var template = thisDoc.querySelector("#chart-pie");
  thatDoc.body.appendChild(template.content.cloneNode(true));
</script>
```
We're creating a new `<template>` which contains a canvas tag and a script block to create our pie chart. The advantage of using a template tag is that any script blocks inside of it will not execute until we clone the contents and add them to the DOM.

On line 27 we pull in the Charts.js library which I've stored in a folder called `lib`. When our index file fetches `chart.html` it will also fetch any `links` or `scripts` so we can include whatever we want.

Lines 30-38 should look familiar. We're grabbing the template and stamping its content onto the document. Notice on line 38 we use `template.content` to access the innards.

Running the above gives us this:

{% img center http://robdodson.s3.amazonaws.com/images/imports-template.jpg 'Imports with Template' %}

Well this is interesting. We're importing an entire pie chart and our index page isn't cluttered with a bunch of code. Unfortunately we don't have much control over where the pie chart ends up. It would be nice if we could turn the contents of the import into a tag and place that wherever. Thankfully Custom Elements let us do just that!

## Using Custom Elements in our Imports
### Sketch 5: [Custom Element](https://github.com/robdodson/webcomponents-sketchbook/tree/master/html-imports/5-custom-element)

I'll say in advance that you might need to read through this section a few times before you fully grok it. We're going to touch on a lot of new stuff and I fully admit that I don't understand it all just yet. Consider it the bonus round :)

Having said that, the final markup for our `index.html` file is going to look like this:

```html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Imports with Custom Elements</title>
    <link rel="import" href="/imports/chart.html">
  </head>
  <body>
    <h1>Quarterly Pokemon Sales</h1>
    <chart-pie></chart-pie>
    <chart-pie></chart-pie>
    <chart-pie></chart-pie>
  </body>
</html>
```
On lines 10-12 we're going to use our new Custom Element, `chart-pie`, which will allow us to produce pie charts wherever we want. The result will look like this:

{% img center http://robdodson.s3.amazonaws.com/images/imports-custom-elements.jpg 'Imports with Custom Elements' %}

Obviously not the most amazing thing ever but from a practical perspective being able to drop a pie chart on your page with one line of HTML is pretty sweet.

To create the `chart-pie` tag we'll need to create a [Custom Element](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/custom/index.html). Custom Elements are new tags with a lifecycle of JavaScript callbacks. Typically they use [Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/) to hide their internal markup and expose attributes and specific styles to the client. [I wrote an article loosely explaining them](http://robdodson.me/blog/2013/03/17/why-web-components/) a while back so take a look at that and also checkout [this talk by Eric Bidelman](http://www.youtube.com/watch?v=fqULJBBEVQE).

Here's what our updated `chart.html` looks like.

```html imports/chart.html
<template id="chart-pie">
  <canvas class="myChart" width="200" height="200"></canvas>
</template>

<script src="/lib/chart.min.js"></script>
<script>
  // thisDoc refers to the "importee", which is chart.html
  var thisDoc = document.currentScript.ownerDocument;

  // thatDoc refers to the "importer", which is index.html
  var thatDoc = document;

  var template = thisDoc.querySelector('#chart-pie');

  // Make sure you extend an existing HTMLElement prototype
  var ChartPieProto = Object.create(HTMLElement.prototype);

  // Setup optional lifecycle callbacks: createdCallback,
  // enteredDocumentCallback, leftDocumentCallback and
  // attributeChangedCallback
  ChartPieProto.createdCallback = function() {
    // Create a ShadowDOM to hold our template content
    var root = this.createShadowRoot();
    var clone = template.content.cloneNode(true);

    // Create the pie chart with Chart.js
    var data = [
      {
        value: 30,
        color:"#F38630"
      },
      {
        value : 50,
        color : "#E0E4CC"
      },
      {
        value : 100,
        color : "#69D2E7"
      }
    ];

    //Get the context of the canvas element we want to select
    var ctx = clone.querySelector('.myChart').getContext('2d');
    var myNewChart = new Chart(ctx).Pie(data);

    // Add the template content + chart to our Shadow DOM
    root.appendChild(clone);
  };

  var ChartPie = thatDoc.register('chart-pie', {prototype: ChartPieProto});
  //var chartPie = new ChartPie();
  //var chartPie = document.createElement('chart-pie');
</script>
```
Let's walk through it piece by piece.


    <template id="chart-pie">
      <canvas class="myChart" width="200" height="200"></canvas>
    </template>

On lines 1-3 we've shortened the `template` down so that it only contains our `canvas` tag. We'll use the Custom Element `createdCallback` to actually instantiate the chart in here.
<br>
    // thisDoc refers to the "importee", which is chart.html
    var thisDoc = document.currentScript.ownerDocument;

    // thatDoc refers to the "importer", which is index.html
    var thatDoc = document;

    var template = thisDoc.querySelector('#chart-pie');

Lines 7-13 should look familar from the last example. We're storing our two documents in variables and querying for the template tag.
<br>
    var ChartPieProto = Object.create(HTMLElement.prototype);

On line 16 we define the prototype for our Custom Element called `ChartPieProto`. This prototype extends the `HTMLElement` prototype which is a requirement for creating a new element.
<br>
    ChartPieProto.createdCallback = function() {
      ...
    };

On line 21 we see the first lifecycle callback, `createdCallback`. The `createdCallback` is run every time the parser hits a new instance of our tag. Therefore we can use it as a kind of Constructor to kickoff the creation of our chart. We'll want to create a new chart instance for each tag so all of our Chart.js code has been moved inside of this callback.
<br>
    var root = this.createShadowRoot();
    var clone = template.content.cloneNode(true);

On lines 23-24 we create a [Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/) to hold the markup for our chart.
<br>
    var data = [
      {
        value: 30,
        color:"#F38630"
      },
      {
        value : 50,
        color : "#E0E4CC"
      },
      {
        value : 100,
        color : "#69D2E7"
      }
    ];
    
    //Get the context of the canvas element we want to select
    var ctx = clone.querySelector('.myChart').getContext('2d');
    var myNewChart = new Chart(ctx).Pie(data);

Lines 27-44 should look familiar. It's the same Chart.js code from before except now we use `querySelector` to find the contents of the template clone and we're using a class for `myChart` instead of an id.
<br>
    root.appendChild(clone);
On line 47 we add the new content to our Shadow DOM.
<br>
    var ChartPie = document.register('chart-pie', {prototype: ChartPieProto});
Line 50 is where we actually register our Custom Element and assign it to the name `chart-pie`. From here you can either place a `<chart-pie></chart-pie>` tag somewhere on your page, or use JavaScript to instantiate an instance and add it to the `document`. This is demonstrated in the comments on lines 51-52. If you refer back to our `index.html` example we just use the `<chart-pie>` tag.

```html index.html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <title>Imports with Custom Elements</title>
    <link rel="import" href="/imports/chart.html">
  </head>
  <body>
    <h1>Quarterly Pokemon Sales</h1>
    <chart-pie></chart-pie>
    <chart-pie></chart-pie>
    <chart-pie></chart-pie>
  </body>
</html>
```
Which produces this:

{% img center http://robdodson.s3.amazonaws.com/images/imports-custom-elements.jpg 'Imports with Custom Elements' %}

## Conclusion

If you've made it this far congrats and thanks for hanging in there! I know that last section was a little crazy but stop for a moment and think about what we just did.

By using an HTML Import we were able to pull in a document which added a new tag to our application. Imagine if *all* of Chart.js was written in this manner. There would be no need for us to write any glue code to generate a chart ever again. Instead we could manipulate the attributes of a `<chart-pie>` tag using something like Angular or any other front-end framework. That would allow us to focus only on the code that matters to our application and leave all that other boilerplate tucked away inside of Custom Elements.

As a matter of fact the Polymer folks have already started doing this. Checkout the [more-elements project on GitHub](https://github.com/Polymer/more-elements) which includes proof of concept wrappers for a number of libraries including Chart.js. Thanks to Scott Miles on [the Polymer Google Group](https://groups.google.com/forum/#!forum/polymer-dev) for pointing that out to me :)

Over the next few months I'll be blogging exclusively about this topic because I think it's really interesting so check back later for more!

Till then make sure to [hit me up on Twitter](http://twitter.com/rob_dodson) if you have any questions or leave a note in the comments. Thanks!
