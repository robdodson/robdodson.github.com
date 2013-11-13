---
layout: post
title: "Repeating Templates in Polymer"
date: 2013-11-12 15:14
comments: true
categories: [Web Components, Polymer, Template, HTML5]
---

I ran into a little issue this afternoon working with templates in Polymer and I wanted to quickly jot down my thoughts in case others bump up against this.

<!-- more -->

## Bindings

Bindings allow you to easily pipe some data into your markup from a JavaScript object of some kind. If you've worked with a library like Mustache or Handlebars before then they should feel familiar.

In Polymer land, the `<template>` tag has been extended so it supports a few handy binding attributes. These include `bind`, `repeat`, `if`, and `ref`.

## How Not to Do It

If you take a look at the Polymer docs on [template bindings](http://www.polymer-project.org/platform/template.html) you'll notice that the binding attribute (`bind`, `repeat`, etc.) is always located on the first template. For instance:

```
<template repeat="&#123;{ collection }}">
  Creates an instance with {{ bindings }} for every element in the array collection.
</template>
```

This lead me to believe that I should write my element like this:

```
<polymer-element name="polymer-letters">
  <template repeat="&#123;{ letter in letters }}">
    &#123;{ letter }}
  </template>
  <script>
    Polymer('polymer-letters', {
      letters: ['a', 'b', 'c']
    });
  </script>
</polymer-element>
```

But unfortunately that does not work <span style="color: grey;">#sadtrombone.</span>

## The Right Way

Polymer uses the first `template` element to create Shadow DOM, so if you want to use a binding **you'll need to nest it *inside* another template.**

Our updated example would look like this:

```
<polymer-element name="polymer-letters">
  <template>
    <template repeat="&#123;{ letter in letters }}">
      &#123;{ letter }}
    </template>
  </template>
  <script>
    Polymer('polymer-letters', {
      letters: ['a', 'b', 'c']
    });
  </script>
</polymer-element>
```

And here it is running on CodePen:

<p data-height="268" data-theme-id="0" data-slug-hash="wxrqf" data-user="robdodson" data-default-tab="html" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/wxrqf'>Polymer Template Bindings</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

I mentioned this to Eric Bidelman and he opened [a ticket to improve the docs](https://github.com/Polymer/docs/issues/191), so keep an eye out for that. Hope this helps some of you that may have been stuck :)