---
layout: post
title: "@font-face doesn't work in the Shadow DOM"
date: 2013-11-19 09:27
comments: true
categories: [Web Components, Shadow DOM, Fonts, HTML5, Polymer]
---

I was building custom elements with Polymer the other day, and I thought it would be cool to include Font Awesome for some sweet icon goodness. Everything was going great, until I switched over to Canary to check my work.

<!-- more -->

{% img center http://robdodson.s3.amazonaws.com/images/polymer-fonts-busted.png 'Busted Polymer Fonts' %}

uh oh...


At first, I was pretty confused. Why was everything working fine in Chrome, but acting crazy in Canary? I did some digging and manged to turn up [this thread](https://groups.google.com/d/msg/polymer-dev/UUwew3x82EU/m9x2qWPi9ZoJ) on the Polymer mailing list.

## The Fix

I had a bit of an "aha moment" when I remembered that the current version of Chrome doesn't actually hide any styles in the Shadow DOM, so my `@font-face` rules were ending up in the global scope. Canary, on the other hand, is encapsulating those styles in a shadow boundary and falling prey to an apparent bug.

A simple workaround is to pull your `@font-face` rules out of the stylesheet for your element, and move them to the top of your import, like so:

```
&lt;style&gt;
  @font-face {
    font-family: 'FontAwesome';
    src: url('../fonts/fontawesome-webfont.eot?v=4.0.3');
    src: url('../fonts/fontawesome-webfont.eot?#iefix&v=4.0.3') format('embedded-opentype'), url('../fonts/fontawesome-webfont.woff?v=4.0.3') format('woff'), url('../fonts/fontawesome-webfont.ttf?v=4.0.3') format('truetype'), url('../fonts/fontawesome-webfont.svg?v=4.0.3#fontawesomeregular') format('svg');
    font-weight: normal;
    font-style: normal;
  }
&lt;/style&gt;

<polymer-element name="semantic-ui-icon" noscript>
  <template>
    <link rel="stylesheet" href="./icon.css">
    <content></content>
  </template>
</polymer-element>
```

I found this approach [in the Polymer documentation](http://www.polymer-project.org/docs/polymer/styling.html#making-styles-global), so I'm hoping it's considered a best practice. You'll also need to do this if you're using `@-webkit-keyframes` rules.

I hope that clears things up for some of you who may have been stuck. I know it took me a couple days to come up with this solution, so I thought it best to go ahead and post about it :)