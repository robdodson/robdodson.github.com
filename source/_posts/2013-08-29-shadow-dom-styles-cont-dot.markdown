---
layout: post
title: "Shadow Dom: Styles (cont.)"
date: 2013-08-29 11:11
comments: true
categories: [HTML5, Shadow DOM, Web Components]
---

In [yesterday's post](/blog/2013/08/28/shadow-dom-styles/) we covered the basics of working with styles in Shadow DOM. But we've only just scratched the surface! Today we'll continue from where we left off and explore how to work with **distributed nodes** and how to poke holes in our components so the outside world can reach in and customize 'em.

<!--more-->

*Before we get started I wanted to thank Eric Bidelman for his [amazing article on styling the Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/). Most of this article is my interpretation of his post. Definitely go read [everything on HTML5 Rocks that pertains to Web Components](http://www.html5rocks.com/en/tutorials/#webcomponents) when you get a chance.*

## Support

In order to try our examples I suggest you use [Chrome Canary](https://www.google.com/intl/en/chrome/browser/canary.html) v31 or greater.

Also make sure you've enabled the following in Chrome's `about:flags`.

√ Experimental Web Platform features<br>
√ Experimental JavaScript<br>

*I believe Shadow DOM is supported in Chrome without experimental flags but we may touch on other Web Component technologies that require them. Better to just turn them on now I think :)*

## Codez!

I've created a sketchbook for this post and future Web Components related stuff. [You can grab the sketchbook on GitHub.](https://github.com/robdodson/webcomponents-sketchbook) For each of the examples that I cover I'll link to the sketch so you can quickly try things out.

## Distributed Nodes

### Sketch 9: [styling-distributed-nodes](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/09-styling-distributed-nodes)

From reading various blog posts I learned that when working with the shadow DOM you should make sure to keep your content separate from your presentation. In other words, if you have a button that's going to display some text, that text should come from the page and not be buried in a shadow DOM template. Contents which come from the page and are added to the shadow DOM using the `<content>` tag are know as **distributed nodes**.

Initially I struggled to understand how it was possible to style these distributed nodes. I kept writing my CSS like this:

```html
<div class="some-shadow-host">
  <button>Hello World!</button>
</div>

<template>
  <style>
    :host {
      ...
    }

    button {
      font-size: 18px;
    }
  </style>
  <content></content>
</template>
```

Thinking that if `button` came from the shadow host I should be able to just style it once it was inside my `<content>` tag. But that's not quite how things work. Instead, distributed nodes need to be styled with the `::content` pseudo selector. This actually makes sense because we might want buttons inside of our shadow template to be styled differently from buttons which appear inside of our `<content>` tags.

Let's look at an exmaple:

```html
<body>
  <div class="widget">
    <button>Distributed Awesomesauce</button>
  </div>

  <template class="widget-template">
    <style>
      ::content > button {
        font-size: 18px;
        color: white;
        background: tomato;
        border-radius: 10px;
        border: none;
        padding: 10px;
      }
    </style>
    <content select=""></content>
  </template>

  <script>
    var host = document.querySelector('.widget');
    var root = host.createShadowRoot();
    var template = document.querySelector('.widget-template');
    root.appendChild(template.content.cloneNode(true));
    template.remove();
  </script>
</body>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-distributed.jpg 'Styling distributed shadow DOM nodes' %}

Here we're pulling in the `button` from our `.widget` shadow host and tossing it into our `<content>` tag. Using the `::content` pseudo selector, we target the `button` as a child with `>` and set our fancy pants styles.

### Syntax Changes

In previous versions of Chrome the syntax for styling distributed nodes looked like this:

```html
content::-webkit-distributed(<selector>)
```
But this style is being deprecated. As of today (August 29, 2013) `::content` is the proper syntax.

## Parts

### Sketch 10: [styling-parts](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/10-styling-parts)

Up to this point we've celebrated the encapsulation benefits of the shadow DOM but sometimes you want to poke a few holes in the shadow boundary so the user can style your component.

Let's say you're creating a sign in form. Inside of your template you've defined the text size for the inputs but you'd like the user to be able to change this so it fits better with her site. Using the `::part` pseudo selector and `part` attribute we can expose any fields we want, thereby giving the user total freedom to override our defaults if they so choose.

```html
<head>
  <style>
    .sign-up::part(username),
    .sign-up::part(password) {
      font-size: 18px;
      border: 1px solid red;
    }

    .sign-up::part(btn) {
      font-size: 18px;
    }
  </style>
</head>
<body>
  <div class="sign-up"></div>

  <template class="sign-up-template">
    <style>
      div[part="username"],
      div[part="password"] {
        font-size: 10px;
      }
    </style>
    <div>
      <input type="text" id="username" part="username" placeholder="username">
    </div>
    <div>
      <input type="password" id="password" part="password" placeholder="password">
    </div>
    <button part="btn">Sign Up!</button>
  </template>

  <script>
    var host = document.querySelector('.sign-up');
    var root = host.createShadowRoot();
    var template = document.querySelector('.sign-up-template');
    root.appendChild(template.content.cloneNode(true));
  </script>
</body>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-part.jpg 'Styling a shadow DOM part' %}

There are three important bits of syntax I want to point out in this example.

The first is *how* we specify that an element is going to be a `part`. We do this through the use of the `part` attribute.

```html
<input type="text" id="username" part="username" placeholder="username">
```

Next we specify a default style for this `part` using an attribute selector inside of our template.

```html
<style>
  div[part="username"],
  div[part="password"] {
    font-size: 10px;
  }
</style>
```

Finally, we reach into the shadow DOM with the `::part` pseudo selector to override the style defaults.

```html
<style>
  .sign-up::part(username),
  .sign-up::part(password) {
    font-size: 18px;
    border: 1px solid red;
  }

  .sign-up::part(btn) {
    font-size: 18px;
  }
</style>
```

### Syntax Changes

In previous versions of Chrome the `part` attribute was known as `pseudo`. A `pseudo` attribute would need to be prefixed with an `x-` and selecting the `pseudo` attribute looked like this:

```html
.foo::x-slider-thumb {
  background-color: blue;
}
```

This syntax is now deprecated and you should use `part` and `::part` instead.

## Variables

### Sketch 11: [styling-with-variables](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/11-styling-with-variables)

Working with all of these hard coded styles is making my inner nerd sad. If you've been spoiled by LESS or SCSS, like I have, then you'll quickly be longing for variables to hold all of your configurable values. Thankfully CSS3 variables are supported by the shadow DOM so let's look at how we can tidy things up a bit.

We'll use our previous example but this time we'll work a little variable magic on it:

```html
<head>
  <style>
    .sign-up {
      var-type-size: 18px;
    }

    .sign-up::part(username),
    .sign-up::part(password) {
      var-borders: 1px solid red;
    }
  </style>
</head>
<body>
  <div class="sign-up"></div>

  <template class="sign-up-template">
    <style>
      .sign-up::part(username),
      .sign-up::part(password) {
        font-size: var(type-size);
        border: var(borders);
      }

      .sign-up::part(btn) {
        font-size: var(type-size);
      }
    </style>
    <div>
      <input type="text" id="username" part="username" placeholder="username">
    </div>
    <div>
      <input type="password" id="password" part="password" placeholder="password">
    </div>
    <button part="btn">Sign Up!</button>
  </template>

  <script>
    var host = document.querySelector('.sign-up');
    var root = host.createShadowRoot();
    var template = document.querySelector('.sign-up-template');
    root.appendChild(template.content.cloneNode(true));
  </script>
</body>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-part.jpg 'Styling a shadow DOM part' %}

We should wind up with the exact same output but things are a bit cleaner now.

Personally I'm not a fan of the CSS3 variable syntax but if you want to go preprocessor free I think it's your best bet.
 
## Inheriting and Resetting Styles

### Sketch 12: [inheriting-and-resetting-styles](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/12-inheriting-and-resetting-styles)

### Applying Author Styles

If your component contains text or other inheritable properties you may want it to match the page that's hosting it. This is where `applyAuthorStyles` comes in to play. By setting `applyAuthorStyles` to `true` you're telling the document that it's ok for the user's CSS to bleed through and affect your component. This is great for things like typography where you want your component to use the same font families and header sizes as the parent document.

```html
<head>
  <style>
    body {
      font-family: Helvetica, Arial, sans-serif;
    }

    h2 {
      text-decoration: underline;
    }

    p {
      font-size: 18px;
      font-weight: bold;
    }
  </style>
</head>
<body>

  <h2>This h2 is NOT in the shadow DOM</h2>
  <p>Neither is this paragraph</p>

  <div class="article">Some interesting article content</div>

  <template class="article-template">
    <h2>An Article Header</h2>
    <p><content></content></p>
  </template>

  <script>
    var host = document.querySelector('.article');
    var root = host.createShadowRoot();
    root.applyAuthorStyles = true;
    // Note that you can also reset styles on a per <shadow> or
    // per <content> basis
    // http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/#toc-shadow-resetstyles
    var template = document.querySelector('.article-template');
    root.appendChild(template.content.cloneNode(true));
    template.remove();
  </script>
</body>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-author-styles.jpg 'Applying author styles' %}

If we take a look at the CSS we can see that the user has directly styled all `h2`'s and `p`'s. Also everything on the page should inherit a `font-family` of Helvetica. By using `applyAuthorStyles` we've allowed the direct styles *and* the inherited `font-family` to bleed through.

### Reseting Inheritance

In some cases we may wish to only allow the direct styles while resetting anything that would be inherited. Using the example above that would mean excluding the `font-family` of Helvetica which is inherited from the `body` style. To reset style inheritance we use the aptly named `resetStyleInheritance` method.

```html
<script>
  var host = document.querySelector('.article');
  var root = host.createShadowRoot();
  root.resetStyleInheritance = true; // <-- get rid of anything inherited
  root.applyAuthorStyles = true;
  var template = document.querySelector('.article-template');
  root.appendChild(template.content.cloneNode(true));
  template.remove();
</script>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-reset-inheritance.jpg 'Reset style inheritance' %}

Now our component reverts to the default `font-family` of Times New Roman while still allowing direct author styles on `h2` and `p`. [Eric Bidelman's great post on Shadow DOM 201 has a handy cheat sheet](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/#style-inherit-cheetsheet) so you can sort out when you want to use `applyAuthorStyles` and when you want to use `resetStyleInheritance`.

## Conclusion

If you've read over [the last post](/blog/2013/08/28/shadow-dom-styles/) and this one then you now know as much about styling the shadow DOM as I do. But we haven't even talk about JavaScript yet! We'll save that for tomorrow's post :) As always if you have questions [hit me up on twitter](http://twitter.com/rob_dodson) or leave a comment. Thanks!
