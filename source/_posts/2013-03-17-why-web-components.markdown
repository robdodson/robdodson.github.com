---
layout: post
title: "Why Web Components?"
date: 2013-03-17 17:48
comments: true
categories: [HTML5, Custom Elements, Web Components]
---

Yesterday I did a post on the HTML5 `<template>` tag which is part of the new [Web Components standard.](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#custom-element-section) I didn't go into a ton of explanation as to why I'm so interested with this spec and I think jumping right to the `<template>` tag was probably pretty confusing for folks who don't have the same needs as me. I want to back up a bit and present a high level overview of Web Components and then illustrate why this is such an important concept.

<!--more-->

## So, what are Web Components?

Web Components are actually a group of standards which all fall under the same umbrella. The W3C does a good job of giving a high level overview in their [Introduction to Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html) article. Here's a quote from the introduction (emphasis is mine):

<p style="text-align: center;">**</p>

"The component model for the Web (also known as Web Components) consists of four pieces designed to be used together **to let web application authors define widgets with a level of visual richness not possible with CSS alone, and ease of composition and reuse not possible with script libraries today.**

These pieces are:

- **templates**, which define chunks of markup that are inert but can be activated for use later;

- **decorators**, which apply templates to let CSS affect rich visual and behavioral changes to documents;

- **custom elements**, which let authors define their own elements, including new presentation and API, that can be used in HTML documents; and

- **shadow DOM** which defines how presentation and behavior of decorators and custom elements fit together in the DOM tree.

Both decorators and custom elements are called **components.**"

-- [Introduction to Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html#custom-element-section)

<p style="text-align: center;">**</p>

In summary there are two kinds of "components": **custom elements** and **decorators**.

**Custom elements** let you define new [HTMLElements](https://developer.mozilla.org/en-US/docs/DOM/HTMLElement), kind of like creating new tags, and **decorators** let you augment existing tags by injecting additional markup and styles via CSS selectors. Both custom elements and decorators leverage **templates** and **shadow DOM** to encapsulate their markup and styles.

To visualize this process think about the iframe that holds a Facebook Like button. The Like button encapsulates all of its markup and styles inside the iframe so it doesn't mess up anything on your page. Unfortunately there are a few downsides to this approach. For starters you have to make an http request to load the content of the iframe, so you would never want to build your entire site out of iframe'd components. Also, the Like button doesn't expose much of an API for you to change its appearance. While that's not a huge deal for a Facebook Like button, it is if you're using a more generic component like a slider or dropdown.

In an ideal world you could have a *framework* of components which hide their markup but expose an API to alter their appearance. Imagine if this:

``` html
<!-- Bootstrap Dropdown -->
<div class="dropdown">
  <ul class="dropdown-menu" role="menu" aria-labelledby="dLabel">
    <li><a tabindex="-1" href="#">Action</a></li>
    <li><a tabindex="-1" href="#">Another action</a></li>
    <li><a tabindex="-1" href="#">Something else here</a></li>
  </ul>
</div>
```

was just this:

``` html
<div is="x-dropdown" class="fancy"></div>
```
and you're most of the way there.

## It's (probably) not for apps. It's for frameworks.

Unless you're trying to write a UI Framework like the next Bootstrap, you can probably ignore the underpinnings of how Web Components are produced. At present we have the `<video>` tag which basically uses all of the technologies listed above. Do you care what its template and shadow DOM look like? Not really. You just want to say:

``` html
<video autoplay="true" controls src="path/to/video">
```

to spit out your awesome player. Think of the cognitive load that you could unburden yourself of if *all* UI Frameworks had the same abilities as `<video>`!

A page might look like this:

``` html
<nav is="x-bootstrap-navbar" sticky></nav>
<section class="container" role="main">
  <!-- mixing Foundation and Bootstrap! -->
  <div is="x-foundation-alert" class="success">Hey you succeeded!</div>
  <table is="x-datatables" ascending src="path/to/data.json"></table>
  <div is="x-bootstrap-pagination"></div>
</section>
<footer is="x-bootstrap-footer"></footer>
```
instead of hundreds of lines of markup rubber stamped over and over again. Even if you take the repetative markup and place it inside a template engine like handlebars you still can't mix and match UI Frameworks like Bootstrap and Foundation without running into some serious risks. Web Components hope to solve both of these problems and open up a much more declarative authoring process.

Hopefully tomorrow I can dig into Custom Elements more and provide some concrete examples knitting all of these concepts together. Thanks for reading!

You should follow me on Twitter [here](http://twitter.com/rob_dodson).