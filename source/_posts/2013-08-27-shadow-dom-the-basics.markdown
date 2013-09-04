---
layout: post
title: "Shadow DOM: The Basics"
date: 2013-08-27 08:52
comments: true
categories: [HTML5, Shadow DOM, Web Components]
---

In our [previous post](/blog/2013/08/26/shadow-dom-introduction/) we introduced the **Shadow DOM** and made the case for why it is important. Today we'll get our hands dirty with some code! By the end of this post you'll be able to create your own encapsulated widgets that pull in content from the outside world and rearrange their bits and pieces to produce something wholly different.

Let's get started!

<!--more-->

## Support

In order to try the examples I suggest you use [Chrome Canary](https://www.google.com/intl/en/chrome/browser/canary.html) v31 or greater.

Also make sure you've enabled the following in Chrome's `about:flags`.

√ Experimental Web Platform features<br>
√ Experimental JavaScript<br>

*I believe Shadow DOM is supported in Chrome without experimental flags but we may touch on other Web Component technologies that require them. Better to just turn them on now I think :)*

## Codez!

I've created a sketchbook for this post and future Web Components related stuff. [You can grab the sketchbook on GitHub.](https://github.com/robdodson/webcomponents-sketchbook) For each of the examples that I cover I'll link to the sketch so you can quickly try things out.

## A Basic Example

### Sketch 0: [basic](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/00-basic)

Let's take a look at a very simple HTML document.

``` html
<body>
  <div class="my-widget">
    <h1>My Widget Header</h1>
    <p>Some my-widget content</p>
  </div>
  <div class="my-other-widget">
    <h1>My Other Widget Header</h1>
    <p>Some my-other-widget content</p>
  </div>
</body>
```

In HTML every element is considered to be a **node**. When you have a group of these nodes nested inside one another it is known as a **node tree**.

{% img center http://robdodson.s3.amazonaws.com/images/tree-chart.svg 'A basic node tree!' %}

What's unique about Shadow DOM is that it allows us to create our own node trees, known as **shadow trees**, that encapsulate their contents and render only what they choose. This means we can inject text, rearrange content, add styles, etc. Here's an example:

``` html
<div class="widget">Hello, world!</div>
<script>
  var host = document.querySelector('.widget');
  var root = host.createShadowRoot();
  root.textContent = 'Im inside yr div!';
</script>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-basic1.jpg 'Our first shadow dom' %}

Using the code above we've replaced the text content of our `.widget` div using a shadow tree. To create a shadow tree we first specify that a node should act as a **shadow host**. In this case we use `.widget` as our shadow host. Then we add a new node to our shadow host, known as a **shadow root**. The shadow root acts as the first node in your shadow tree and all other nodes descend from it.

If you were to inspect this element in the Chrome Dev Tools (and you had [Show Shadow DOM](/blog/2013/08/26/shadow-dom-introduction#the-shadows) turned on) it would look something like this:

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-basic-inspect.jpg 'Inspecting our first shadow dom' %}

Do you see how `#document-fragment` is grayed out? That's our shadow root. **The takeaway is that the content inside of the shadow host is _not_ rendered. Instead, the content inside of the shadow root is what gets rendered.**

This is why when we run our example we see `Im inside yr div!` instead of `Hello, world!`

We can visualize this process in another graph:

{% img center https://dvcs.w3.org/hg/webcomponents/raw-file/tip/assets/images/tree-of-trees.svg 'Shadow tree graph' %}

Let's do another example to help it all sink it.

## A Basic Example (cont)

### Sketch 1: [basic-cont](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/01-basic-cont)

```html
<body>
  <div class="widget">Hello, world!</div>
  <script>
    var host = document.querySelector('.widget');
    var root = host.createShadowRoot();
    
    var header = document.createElement('h1');
    header.textContent = 'A Wild Shadow Header Appeared!';
    
    var paragraph = document.createElement('p');
    paragraph.textContent = 'Some sneaky shadow text';

    root.appendChild(header);
    root.appendChild(paragraph);
  </script>
</body>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-basic-cont1.jpg 'A slightly more advanced shadow dom sketch' %}

Here I'm taking our previous example and just adding two additional elements to it. I've done this to illustrate that working with the Shadow DOM is really not so different from working with the regular DOM. You still create elements and use `appendChild` or `insertBefore` to add them to a parent. In the Chrome Dev Tools it looks like this:

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-basic-cont-inspect.jpg 'Inspecting our shadow dom' %}

Just like before, the content in the *shadow host*, `Hello, world!`, is not rendered. Instead the content in the *shadow root* is rendered.

"That's easy enough," you might say. "But what if I actually *want* the content in my shadow host to render?" Well dear reader, you'll be happy to know that's not only possible but it's actually one of the killer features of Shadow DOM. Let's keep going!

## Content

### Sketch 2: [content](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/02-content)

In our last two examples we've completely replaced the content in our shadow hosts with whatever was inside our shadow root. While this is a neat trick, in practice, it's not very useful. What would be really great is if we could take the **content** from our shadow host and then use the structure of the shadow root for **presentation**. Separating the content from the presentation like this allows us to be much more flexible with how our page actually renders.

First thing's first, to use the content in our shadow host we're going to need to employ the new `<content>` tag. Here's an example:

```html
<body>
  <div class="pokemon">
    Jigglypuff
  </div>

  <template class="pokemon-template">
    <h1>A Wild <content></content> Appeared!</h1>
  </template>

  <script>
    var host = document.querySelector('.pokemon');
    var root = host.createShadowRoot();
    var template = document.querySelector('.pokemon-template');
    root.appendChild(template.content.cloneNode(true));
  </script>
</body>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-content1.jpg 'Mixing content and presentation' %}

If you've been following along this should look familiar. Using the `<content>` tag we've created an **insertion point** that **projects** the text from the `.pokemon` div so it appears within our shadow `<h1>`. Insertion points are very powerful because they allow us to change the order in which things render without physically altering the source. It also means that we can be selective about what gets rendered.

You may have noticed that we're using a [template tag](/blog/2013/03/16/html5-template-tag-introduction/) instead of building our Shadow DOM entirely in JavaScript. I've found that using `<template>` tags makes the process of working with the Shadow DOM much easier.

Let's look at a more advanced example to demonstrate how to work with multiple insertion points.

## Selects

### Sketch 3: [selects](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/03-selects)

```html
<body>
  <div class="bio">
    <span class="first-name">Rob</span>
    <span class="last-name">Dodson</span>
    <span class="city">San Francisco</span>
    <span class="state">California</span>
    <p>
      I specialize in Front-End development (HTML/CSS/JavaScript) with a touch of Node and Ruby sprinkled in. I’m also a writer and occasional daily blogger. Though I’m originally from the South, these days I live and work in beautiful San Francisco, California.
    </p>
  </div>

  <template class="bio-template">
    <dl>
      <dt>First Name</dt>
      <dd><content select=".first-name"></content></dd>
      <dt>Last Name</dt>
      <dd><content select=".last-name"></content></dd>
      <dt>City</dt>
      <dd><content select=".city"></content></dd>
      <dt>State</dt>
      <dd><content select=".state"></content></dd>
    </dl>
    <p><content select=""></content></p>
  </template>

  <script>
    var host = document.querySelector('.bio');
    var root = host.createShadowRoot();
    var template = document.querySelector('.bio-template');
    root.appendChild(template.content);
  </script>
</body>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-selects1.jpg 'Using content selects' %}

In this example we're building a very simple biography widget. Because each definition field needs specific content we have to tell the `<content>` tag to be selective about where things are inserted. To do this we use the `select` attribute. The `select` attribute uses CSS selectors to pick out which items it wishes to display.

For instance, `<content select=".last-name">` looks inside the shadow host for any element with a matching class of `.last-name`. If it finds a match it will render its content inside of the Shadow DOM.

### Changing Order

As I mentioned before, insertion points allow us to change the rendering order of our presentation without needing to modify the structure of our content. **Remember, the content is what lives in the shadow host and the presentation is what lives in the shadow root/shadow DOM.** A good example would be to swap the rendering order of the first and last names.

```html
<template class="bio-template">
  <dl>
    <dt>Last Name</dt>
    <dd><content select=".last-name"></content></dd>
    <dt>First Name</dt>
    <dd><content select=".first-name"></content></dd>
    <dt>City</dt>
    <dd><content select=".city"></content></dd>
    <dt>State</dt>
    <dd><content select=".state"></content></dd>
  </dl>
  <p><content select=""></content></p>
</template>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-selects2.jpg 'Changing the order of our selects' %}

By simply changing the structure of our `template` we've altered the presentation but the order of the content remains the same. To understand what I mean, take a look at the Chrome Dev Tools.

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-selects-inspect.jpg 'Changing the order of our selects' %}

As you can see, `.first-name` is still the first child in the shadow host. But we've made it appear as if it comes after `.last-name`. We did just just by changing the order of our insertion points. That's pretty powerful when you think about it.

### Greedy Insertion Points

You may have noticed at the end of the `.bio-template` we have a `<content>` tag with an empty string inside of its `select` attribute.

```html
<p><content select=""></content></p>
```

This is considered a wildcard selection and it will grab any content in the shadow host that is left over. The following three selections are all equivalent:

```html
<content></content>
<conent select=""></conent>
<content select="*"></content>
```

As an experiment let's move this wildcard selection to the top of our `template`.

```html
<template class="bio-template">
  <p><content select=""></content></p>
  <dl>
    <dt>Last Name</dt>
    <dd><content select=".last-name"></content></dd>
    <dt>First Name</dt>
    <dd><content select=".first-name"></content></dd>
    <dt>City</dt>
    <dd><content select=".city"></content></dd>
    <dt>State</dt>
    <dd><content select=".state"></content></dd>
  </dl>
</template>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-selects3.jpg 'Overlly greedy selects' %}

You'll notice that it completely changes the presentation of our widget by moving all content inside of the `<p>` tag. This is because selections are greedy and items may only be selected once. Since we have a wildcard selection at the top of our template it grabs all of the content from the shadow host and doesn't leave anything for the other `selects`.

Dominic Cooney ([@coonsta](http://twitter.com/coonsta)) does a great job of describing this in his post on [Shadow DOM 101](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom/) in which he compares the selection process to party invitations.

{% blockquote %}
The content element is the invitation that lets content from the document into the backstage Shadow DOM rendering party. These invitations are delivered in order; who gets an invitation depends on to whom it is addressed (that is, the select attribute.) Content, once invited, always accepts the invitation (who wouldn’t?!) and off it goes. If a subsequent invitation is sent to that address again, well, nobody is home, and it doesn’t come to your party.
{% endblockquote %}

Mastering selects and insertion points can be pretty tricky so Eric Bidelman ([@ebidel](http://twitter.com/ebidel)) created an [insertion point visualizer](http://html5-demos.appspot.com/static/shadowdom-visualizer/index.html) to help illustrate the concepts.

He also created this handy video explaining it :)

<iframe width="420" height="315" src="//www.youtube.com/embed/qnJ_s58ubxg" frameborder="0" allowfullscreen></iframe>

## Conclusion

We still have a lot more to talk about but for today let's wrap things up. Tomorrow we'll dig into CSS style encapsulation and later JavaScript and user interaction. As always, if you have questions [hit me up on twitter](http://twitter.com/rob_dodson) or leave a comment. Thanks!

## [Continue to Shadow DOM: Styles &rarr;](/blog/2013/08/28/shadow-dom-styles/)