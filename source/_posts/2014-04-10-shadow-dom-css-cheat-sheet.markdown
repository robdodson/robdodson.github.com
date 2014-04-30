---
layout: post
title: "Shadow DOM CSS Cheat Sheet"
date: 2014-04-10 22:56
comments: true
categories: [Shadow DOM, Web Components, Polymer, CSS]
---

This guide is my attempt to track the progress of all the new CSS selectors which affect the Shadow DOM. I've written this from the perspective of someone who uses [Polymer](http://polymer-project.org) so in a few places I point out polyfill features like `shim-shadowdom` and `polyfill-next-selector`. But the selectors themselves are all native and comply to [the current draft spec](http://drafts.csswg.org/css-scoping/).

<!-- more -->

Found a bug? [Submit a pull request!](https://github.com/robdodson/robdodson.github.com/blob/source/source/_posts/2014-04-10-shadow-dom-css-cheat-sheet.markdown)

Follow [@rob_dodson on the twitters](http://twitter.com/rob_dodson)

<br>
<br>

<h2><a href="#shadow" id="shadow" class="no-underline">::shadow</a></h2>

Selects shadow trees that are one level deep inside of an element. Will need to be combined with `shim-shadowdom` directive if used outside of a Polymer element in browsers that lack support for the native selector.

```
x-foo::shadow h1 {
  color: red;
}
```
[Try it on CodePen](http://codepen.io/robdodson/pen/HeLEb) | [Read the Spec](http://drafts.csswg.org/css-scoping/#selectordef-shadow)

<table class="plain">
  <thead>
    <tr>
      <th>Support Type</th>
      <th>Chrome</th>
      <th>Firefox</th>
      <th>Internet Explorer</th>
      <th>Safari</th>
      <th>Opera</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-border">Polyfill</td>
      <td class="supported">Yes</td>
      <td class="supported">Yes</td>
      <td class="supported">10+</td>
      <td class="supported">6+</td>
      <td class="supported">Yes</td>
    </tr>
    <tr>
      <td>Native</td>
      <td class="supported">35</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
    </tr>
  </tbody>
</table>

<br>
<br>

<h2><a href="#deep" id="deep" class="no-underline">/deep/</a></h2>

Selects shadow trees that are N levels deep inside of an element. Will need to be combined with `shim-shadowdom` directive if used outside of a Polymer element in browsers that lack support for the native selector.

```
x-foo /deep/ h1 {
  color: red;
}
```
[Try it on CodePen](http://codepen.io/robdodson/pen/wraDn/) | [Read the Spec](http://drafts.csswg.org/css-scoping/#selectordef-deep)

<table class="plain">
  <thead>
    <tr>
      <th>Support Type</th>
      <th>Chrome</th>
      <th>Firefox</th>
      <th>Internet Explorer</th>
      <th>Safari</th>
      <th>Opera</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-border">Polyfill</td>
      <td class="supported">Yes</td>
      <td class="supported">Yes</td>
      <td class="supported">10+</td>
      <td class="supported">6+</td>
      <td class="supported">Yes</td>
    </tr>
    <tr>
      <td>Native</td>
      <td class="supported">35</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
    </tr>
  </tbody>
</table>

<br>
<br>

<h2><a href="#host" id="host" class="no-underline">:host</a></h2>
Selects a shadow host element. May contain additional identifiers in parenthesis.

```
:host(.fancy) {
  display: inline-block;
  background: purple;
}
```
[Try it on CodePen](http://codepen.io/robdodson/pen/rDuyJ/) | [Read the Spec](http://drafts.csswg.org/css-scoping/#selectordef-host0)

<table class="plain">
  <thead>
    <tr>
      <th>Support Type</th>
      <th>Chrome</th>
      <th>Firefox</th>
      <th>Internet Explorer</th>
      <th>Safari</th>
      <th>Opera</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-border">Polyfill</td>
      <td class="supported">Yes</td>
      <td class="supported">Yes</td>
      <td class="supported">10+</td>
      <td class="supported">6+</td>
      <td class="supported">Yes</td>
    </tr>
    <tr>
      <td>Native</td>
      <td class="supported">35</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
    </tr>
  </tbody>
</table>

<br>
<br>

<h2><a href="#host-context" id="host-context" class="no-underline">:host-context</a></h2>
Selects a shadow host based on a matching parent element.

```
:host-context(.blocky) {
  display: block
  background: red;
}
```
[Try it on CodePen](http://codepen.io/robdodson/pen/ftpoG/) | [Read the Spec](http://drafts.csswg.org/css-scoping/#selectordef-host-context)

<table class="plain">
  <thead>
    <tr>
      <th>Support Type</th>
      <th>Chrome</th>
      <th>Firefox</th>
      <th>Internet Explorer</th>
      <th>Safari</th>
      <th>Opera</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-border">Polyfill</td>
      <td class="supported">Yes</td>
      <td class="supported">Yes</td>
      <td class="supported">10+</td>
      <td class="supported">6+</td>
      <td class="supported">Yes</td>
    </tr>
    <tr>
      <td>Native</td>
      <td class="supported">35</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
    </tr>
  </tbody>
</table>

<br>
<br>

<h2><a href="#content" id="content" class="no-underline">::content</a></h2>
Selects distributed nodes inside of an element. Needs to be paired with `polyfill-next-selector` for browsers that do not support the native selector.

```
::content h1 {
  color: red;
}
```
[Try it on CodePen](http://codepen.io/robdodson/pen/FokEw/) | [Read the Spec](http://drafts.csswg.org/css-scoping/#selectordef-content)

<table class="plain">
  <thead>
    <tr>
      <th>Support Type</th>
      <th>Chrome</th>
      <th>Firefox</th>
      <th>Internet Explorer</th>
      <th>Safari</th>
      <th>Opera</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no-border">Polyfill</td>
      <td class="supported">Yes</td>
      <td class="supported">Yes</td>
      <td class="supported">10+</td>
      <td class="supported">6+</td>
      <td class="supported">Yes</td>
    </tr>
    <tr>
      <td>Native</td>
      <td class="supported">35</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
      <td>?</td>
    </tr>
  </tbody>
</table>



