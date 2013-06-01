---
layout: post
title: "Introduction to Custom Elements"
date: 2013-03-19 01:21
comments: true
categories: [HTML5, Web Components, Custom Elements, Shadow DOM]
---

[In yesterday's post](http://robdodson.me/blog/2013/03/17/why-web-components/) I made the case for Web Components so today let's dive back in and look at Custom Elements.

<!--more-->

## So, what are Custom Elements?

The [W3C Introduction to Web Components](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/explainer/index.html) defines Custom Elements as follows:

> custom elements let authors define their own elements, including new presentation and API, that can be used in HTML documents

To translate that a little bit, consider that every HTML tag inherits from [HTMLElement](https://developer.mozilla.org/en-US/docs/DOM/HTMLElement). Often in the W3C specs they'll shorthand that and refer to items in the DOM as just "element" or "elements". So what this is basically saying is that the custom elements spec lets you define *new* HTMLElements, which is kind of like saying you get to define new HTML tags. Technically it has always been possible to create your own HTML tags by shimming the browser, this is in fact how [HTML5shim](need link) makes legacy browsers recognize HTML5 tags, but these tags weren't really first class citizens. Which is to say, they *could* work as containers but if they needed to do anything else like spit out some custom DOM (think the `<video>` tag or an `<input>` field) then that wasn't going to just magically happen. You'd have to write some custom javascript for that and somehow trigger it on the page.

Custom elements solves these issues by letting you define new markup which can then have a whole lifecycle of events, unique styles that are only scope to the element and additional (hidden) HTML markup.

## Defining a new Custom Element

It's probably easier to grok all of this if you just see some example code. Let's look at the proposed markup for defining a new custom element.

``` html
<element extends="button" name="x-dropdown-button">
  …
</element>
```
The W3C definition of this is painfully muddled:

"The element element defines a custom element. It specifies the type of element it's a refinement of using the extends attribute."

To translate, you create an element tag, give it a unique identifier with `name` and specify which tag you're extending. All custom elements *must* extend an existing tag and ideally it should be a tag that they closely relate to. In this case we're creating a new custom element called `x-dropdown-button` which extends `<button>`. The docs suggest that if you don't have a closely related tag that you should just extend `<div>` or `<span>` since (at a minimum) you know that your element is related to a container of some sort. As a fallback legacy browsers will then use the original tag and at least render something that is close to your intent. Maybe IE8 won't get a `x-dropdown-button` but at least it'll get a `button`.

## Defining Custom Elements with Templates

If you recall I said that Custom Elements can define additional markup, similar to the `<video>` tag or an `<input>` field. This markup is specified through the use of a `<template>` tag. For example:

``` html
<element extends="button" name="x-dropdown-button">
  <template>
    <content></content>
    <div class="dropdown">
      <ul class="dropdown-menu">
        <li><a tabindex="-1" href="#">Action</a></li>
        <li><a tabindex="-1" href="#">Another action</a></li>
        <li><a tabindex="-1" href="#">Something else here</a></li>
      </ul>
    </div>
  </template>
</element>
```

If a custom element includes a template then it will be cloned into that element's **shadow DOM** when the browser parses the tag. We haven't talked much about shadow DOM yet but for now just think of it as markup that's *rendered* on the page but not inspectable by the client.

Any content that would normally live inside of the extended tag ends up inside of the `<content>` tag in the template. For instance if we extended `<button>Hello World</button>` then "Hello World" would end up in the `<content>` tag.

In addition to markup the template can include styles that are scoped only to the custom element. Here's the same example from above with the addition of a scoped style tag.

```html
<element extends="button" name="x-dropdown-button">
  <template>
    <style scoped>
      .dropdown-menu {
        font-size: 12px;
        color: #CCC;
      }
    </style>
    <content></content>
    <div class="dropdown">
      <ul class="dropdown-menu">
        <li><a tabindex="-1" href="#">Action</a></li>
        <li><a tabindex="-1" href="#">Another action</a></li>
        <li><a tabindex="-1" href="#">Something else here</a></li>
      </ul>
    </div>
  </template>
</element>
```
The CSS used in the scoped style tag will not affect anything else on the page so you can have your own `.dropdown-menu` classes and they won't conflict with the component. The component author *can* expose style hooks for the consumer to alter the appearance of the element through the use of the new `pseudo` attribute.

```html
<element extends="button" name="x-dropdown-button">
  <template>
    <style scoped>
      .dropdown-menu {
        font-size: 12px;
        color: #CCC;
      }
    </style>
    <content></content>
    <div class="dropdown">
      <ul class="dropdown-menu">
        <li pseudo="x-dropdown-item"><a tabindex="-1" href="#">Action</a></li>
        <li pseudo="x-dropdown-item"><a tabindex="-1" href="#">Another action</a></li>
        <li pseudo="x-dropdown-item"><a tabindex="-1" href="#">Something else here</a></li>
      </ul>
    </div>
  </template>
</element>
```

elsewhere in a style.css you could say:

```css
x-dropdown-button::x-dropdown-item {
  background-color: blue;
  height: 40px;
}
```

## Using Custom Elements in Markup

Using a custom element is really straightforward thanks to the new `is` attribute. To use our new dropdown-button from above would just be:

```html
<button is="x-dropdown-button">Hey I'm a dropdown!</button>
```
Easy, right?

You can also create custom element in script like so:

```js
var btn = document.createElement("x-dropdown-button");
alert(btn.outerHTML); // will display '<button is="x-dropdown-button"></button>'
```

Also you can specify a `constructor` attribute and then use it to create new instances of your element.

```html
<element extends="button" name="x-dropdown-button" constructor="DropdownButton">
  …
</element>

var btn = new DropdownButton();
document.body.appendChild(b);
```

Additionally you can add methods and properties to this constructor and call them later:

```html
<element extends="button" name="x-dropdown-button" constructor="DropdownButton">
  <script>
  DropdownButton.prototype.open = function () {
    alert("Hey I'm open!");
  };
  DropdownButton.prototype.close = function () {
    alert("Aw dang I'm closed...");
  };
  </script>
</element>

…

<script>
    var btn = new DropdownButton();
    document.body.appendChild(btn);
    b.open(); // alerts "Hey I'm open!"
</script>
```

Constructors are attached to the `window` object, but if you don't want to pollute that scope you can use the `generatedConstructor` property:

```html
<element extends="table" name="x-chart">
  <script>
    // …
    var DropdownButton = this.generatedConstructor;
    DropdownButton.prototype.open = function() { /* … */ };
    // …
  </script>
</element>
```

## Support

Unfortunately, at least to my knowledge, custom elements are not currently supported in any major browsers. However Chrome has added support for 2 of the 4 Web Components standards in their last two releases so I can't help but think that custom elements are right around the corner. One thing I didn't touch on very much in this post was shadow DOM which currently **is** supported in Chrome so tomorrow I'll talk about that technology. Till then!

You should follow me on Twitter [here](http://twitter.com/rob_dodson).