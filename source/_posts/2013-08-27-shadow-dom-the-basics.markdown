---
layout: post
title: "Shadow DOM: The Basics"
date: 2013-08-27 08:52
comments: true
published: false
categories: [HTML5, Shadow DOM, Web Components]
---

In our [previous post](/blog/2013/08/26/shadow-dom-introduction/) we introduced the **Shadow DOM** and made the case for why it is important. Today we'll get our hands dirty with some code! By the end of this post you'll be able to create your own encapsulated widgets which pull in content from the outside world and rearrange their bits and pieces to produce something wholly different.

Let's get started!

<!--more-->

## Support

In order to try our examples I suggest you use [Chrome Canary](https://www.google.com/intl/en/chrome/browser/canary.html) v31 or greater.

Also make sure you've enabled the following in Chrome's `about:flags`.

√ Experimental Web Platform features<br>
√ Experimental JavaScript<br>

*I believe Shadow DOM is supported in Chrome without experimental flags but we may touch on other Web Component technologies which require them. Better to just turn them on now I think :)*

## Codez!

I've created a sketchbook for this post and future Web Components related stuff. [You can grab the sketchbook on GitHub.](https://github.com/robdodson/webcomponents-sketchbook) For each of the examples that I cover I'll link to the sketch so you can quickly try things out.

## A Basic Example

### Sketch 0: [Basic](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/00-basic)

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

## [Continue to Shadow DOM: Styles &rarr;](#)