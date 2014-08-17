---
layout: post
title: "HTML5 Template Tag: Introduction"
date: 2013-03-16 09:51
comments: true
categories: [HTML5, Template, Web Components]
---

If you've been following the Google Chrome release announcements you may have noticed some interesting terms popping up as of late. Phrases like "Shadow DOM", "Custom Elements" and "Template". These are all part of a new standard referred to as Web Components and the latest versions of Chrome are starting to implement them. Basically a Web Component is an encapsulated bit of markup, styles and script that allow you to reuse widgets across your page without worrying about accidental collisions. For a practical example consider Twitter Bootstrap and its myriad of buttons, dropdowns and badges. Bootstrap comes with *a lot* of CSS which means, right out of the gate, there are a ton of class names which you probably shouldn't use for your own elements. Want to define a class of `.container`? Too bad, Bootstrap already uses that name. Want to upgrade Bootstrap in a few months? OK, but you need to double check that none of their new classes conflict with any of yours. Kind of a pain, right? Wouldn't it be awesome if you could use all the little widgets from Bootstrap and not worry at all about possible collisions? Well, that's what Web Components are trying to solve and so today I want to look at one part of the Web Components standard: The Template Tag.

<!--more-->

## Feature Detection

The &lt;template&gt; tag is currently supported in Chrome, Firefox, Opera and Safari 8+. You can see the full bracket of supported browsers over on [caniuse](http://caniuse.com/#feat=template). Here's a little feature dection script I stole from the [HTML5Rocks article](http://www.html5rocks.com/en/tutorials/webcomponents/template/) on `<template>`.

``` html
<h3 class="template-detection">Does your browser support &lt;template&gt;: </h3>

<script>
  function supportsTemplate() {
    return 'content' in document.createElement('template');
  }

  $('.template-detection').append(supportsTemplate().toString().toUpperCase());
</script>
```

<h3 class="template-detection">Does your browser support &lt;template&gt;: </h3>

<script src="//code.jquery.com/jquery-2.1.1.min.js"></script>
<script>
  function supportsTemplate() {
    return 'content' in document.createElement('template');
  }

  $('.template-detection').append(supportsTemplate().toString().toUpperCase());
</script>

## Our first template

There's a great [HTML5Rocks article](http://www.html5rocks.com/en/tutorials/webcomponents/template/) on the subject of the `template` tag and I'm going to steal some of their examples. 
Let's start by making a template for an image placeholder. We're going to use the [hhhhold](http://hhhhold.com/) image service which will load in a random image from [ffffound.com](http://ffffound.com). The markup for our template will be pretty simple:

``` html
<template id="hhhhold-template">
  <img src="" alt="random hhhhold image">
  <h3 class="title"></h3>
</template>

<script>
  var template = document.querySelector('#hhhhold-template');
  template.content.querySelector('img').src = 'http://hhhhold.com/350x200';
  template.content.querySelector('.title').textContent = 'Random image from hhhhold.com'
  document.body.appendChild(document.importNode(template.content, true));
</script>
```

Which would render something like this (remember to view in a browser that supports template):

<div id="hhhold-container"></div>
<template id="hhhhold-template">
  <img src="" alt="random hhhhold image">
  <h3 class="title"></h3>
</template>

<script>
  var template = document.querySelector('#hhhhold-template');
  template.content.querySelector('img').src = 'http://hhhhold.com/350x200';
  template.content.querySelector('.title').textContent = 'Random image from hhhhold.com'
  document.querySelector('#hhhold-container').appendChild(
    document.importNode(template.content, true)
  );
</script>

If you've worked with client-side template libraries like underscore or handelbars the above should look familiar to you. Where underscore and handelbars take advantage of putting their templates inside of `<script>` tags and changing the `type` to something like `text/x-handlebars-template`, the `<template>` tag doesn't need to, because it's actually part of the HTML DOM spec. There are pros and cons to this approach.

### Pros

- The content of the template is inert, scripts won't run, images won't load, audio won't play, etc. This means you can have `<img>` and `<script>` tags whose `src` attributes haven't been defined yet.
- The child nodes of a template are hidden from selectors like `document.getElementById()` and `querySelector()` so you won't accidentally select them.
- You can place the `<template>` pretty much anywhere on the page and grab it later.

### Cons

- You can't precompile the template into a JS function like you can with other libraries like handlebars.
- You can't preload the assets referenced by a template (images, sounds, css, js).
- You can't nest templates inside of one another and have it automagically work. If a template contains another template you'll have to activate the child, then activate the parent.
- There's no support for data interpolation (i.e. filling a template with values from a JS object).

Given that list of cons you might say "Well why would I ever bother with the `<template>` tag if something like handlebars gives me way more power?" That's a great question because by itself the `<template>` tag doesn't seem so impressive.

Its saving grace lies in the fact that it is part of the DOM, whereas all other template libraries are actually just pushing around Strings. That makes them vulnerable to XSS and requires weird hacks to prevent the browser from rendering their content. While features like data interpolation are pretty crucial, they can be fixed by the next generation of template libraries, in fact [Polymer's](polymer-project.org) templating has already added this back in. Which leads to the bigger point: combining templates with Shadow DOM and Custom Elements gives us the future component model for the web, and that's why I'm truly excited to use them.

Think back to the story about Bootstrap that I told at the beginning of this post. If all the markup for a Bootstrap button lived inside a template tag then we'd be one step closer to having a nice encapsulated widget. The next step would be to isolate the styles associated with the button. But I'll save that for tomorrow's post :)

You should follow me on Twitter [here](http://twitter.com/rob_dodson).
