---
layout: post
title: "The Cat and the Hat CSS Selectors"
date: 2013-11-15 11:08
comments: true
categories: [CSS, Shadow DOM, Web Components, Polymer]
---

One of the trickier aspects of encapsulating Shadow DOM CSS is figuring out how much access the parent document should have. Initially it was thought that the Shadow DOM's author would decide which elements could be exposed for styling [by using `part` attributes](/blog/2013/08/29/shadow-dom-styles-cont-dot#parts), but it seems like that might be too limiting. The thinking now is that the shadow boundary should prevent *accidental* styling of the shadow DOM, but allow intentional styles. That's where the new "cat" and "hat" CSS selectors come in.

<!-- more -->

## Support <a href="#" id="support"></a>

In order to try the examples I suggest you use [Chrome Canary](https://www.google.com/intl/en/chrome/browser/canary.html) v33 or greater.

Also make sure you've enabled **Experimental Web Platform features** in Chrome's `chrome://flags`.

## The Hat <a href="#" id="the-hat"></a>

The hat selector, a single caret ( ^ ), allows the parent document to pierce the **upper shadow boundary** and style elements that are one shadow root deep. If you have an element that only has one shadow root you can style pretty much anything inside of it using the hat.

<p data-height="268" data-theme-id="0" data-slug-hash="EhIax" data-user="robdodson" data-default-tab="css" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/EhIax'>Shadow DOM "Hat" CSS selector</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

## The Cat <a href="#" id="the-cat"></a>

The cat, a double caret ( ^^ ) is much more powerful. It allows you to pierce every layer of the shadow DOM, so if you have shadow DOM within shadow DOM (a common occurrence when you nest custom elements) you can style all of them at once.

<p data-height="315" data-theme-id="0" data-slug-hash="wFqJg" data-user="robdodson" data-default-tab="css" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/wFqJg'>Shadow DOM "Cat" CSS selector</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

## Styling Native Elements <a href="#" id="styling-native-elements"></a>

[@Volker_E asked](https://twitter.com/Volker_E/status/401202275009310722) if the cat and hat selectors could be used to style the shadow DOM of native elements like `<video>`. As it turns out that *does* work, which is pretty cool.

<p data-height="268" data-theme-id="0" data-slug-hash="iaJHd" data-user="robdodson" data-default-tab="css" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/iaJHd'>Shadow DOM "Cat" and "Hat" CSS selectors</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>