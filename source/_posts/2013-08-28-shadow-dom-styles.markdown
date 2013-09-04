---
layout: post
title: "Shadow DOM: Styles"
date: 2013-08-28 10:56
comments: true
categories: [HTML5, Shadow DOM, Web Components]
---

[Yesterday's post](/blog/2013/08/27/shadow-dom-the-basics/) was all about coding and structuring your first Shadow DOM elements. But I'm sure most of you were wondering, how do we style these things?!

The use of CSS in Shadow DOM is an interesting and large topic. So large, in fact, that I'm going to split it up over the next couple of posts. Today we'll learn the basics of using CSS within the **shadow boundary**, and also how to style our **shadow hosts**.

<!--more-->

*Before we get started I wanted to thank Eric Bidelman for his [amazing article on styling the Shadow DOM](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/). Most of this article is my interpretation of his post. Definitely go read [everything on HTML5 Rocks that pertains to Web Components](http://www.html5rocks.com/en/tutorials/#webcomponents) when you get a chance.*

## Support

In order to try the examples I suggest you use [Chrome Canary](https://www.google.com/intl/en/chrome/browser/canary.html) v31 or greater.

Also make sure you've enabled the following in Chrome's `about:flags`.

√ Experimental Web Platform features<br>
√ Experimental JavaScript<br>

*I believe Shadow DOM is supported in Chrome without experimental flags but we may touch on other Web Component technologies that require them. Better to just turn them on now I think :)*

## Codez!

I've created a sketchbook for this post and future Web Components related stuff. [You can grab the sketchbook on GitHub.](https://github.com/robdodson/webcomponents-sketchbook) For each of the examples that I cover I'll link to the sketch so you can quickly try things out.

## Style Encapsulation

### Sketch 4: [style-encapsulation](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/04-style-encapsulation)

Astute readers probably noticed that I used a new term in the introduction for this post. That term is **shadow boundary** and it refers to the barrier that separates the regular DOM (the "light" DOM) from the shadow DOM. One of the primary benefits of the shadow boundary is that it prevents styles that are in the main document from leaking into the shadow DOM. This means that even though you might have a selector in your main document for all `<h3>` tags, that style will not be applied to your shadow DOM element unless you specifically allow it.

Examples? Yes, let's.

```html
<body>
    <style>
      button {
        font-size: 18px;
        font-family: cursive;
      }
    </style>
    <button>I'm a regular button</button>
    <div></div>

    <script>
      var host = document.querySelector('div');
      var root = host.createShadowRoot();
      root.innerHTML = '<style>button { font-size: 24px; color: blue; } </style>' +
                       '<button>I\'m a shadow button</button>'
    </script>
  </body>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-styles1.jpg 'Shadow DOM style encapsulation' %}

Here we have two buttons. One is in the regular DOM, and the other is in the shadow DOM. You'll notice that the style tag at the top of the page instructs all buttons to use a cursive font and to have a `font-size` of 18px.

```html
<style>
  button {
    font-size: 18px;
    font-family: cursive;
  }
</style>
```
Because of the shadow boundary, the second button ignores this style tag and instead implements its own look. We never specifically override the `font-family` to change it back to sans serif, it just uses the typical browser defaults.

Keep in mind that the shadow boundary also protects the main document from the shadow DOM. You'll notice that our shadow button has a `color` of blue. But the button in the original document maintains its default appearance.

This kind of **encapsulation** is pretty amazing. For years we've struggled with style sheets that always seem to get bigger and bigger. Over time it can be difficult to add new styles because you're worried you'll break something elsewhere on the page. The style boundaries provided to us by the shadow DOM means that we can finally start to think about our CSS in a more local, component specific way.

## Styling :host

### Sketch 5: [styling-host](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/05-styling-host)

I often think of the shadow host as if it's the exterior of a building. Inside there's all the inner workings of my widget and outside there should be a nice facade. In many cases you'll want to apply some style to this exterior and that's where the `:host` selector comes into play.

```html
<body>
  <style>
    .widget {
      text-align: center;
    }
  </style>

  <div class="widget">
    <p>Hello World!</p>
  </div>

  <script>
    var host = document.querySelector('.widget');
    var root = host.createShadowRoot();
    root.innerHTML = '<style>' +
                     ':host {' +
                     '  border: 2px dashed red;' +
                     '  text-align: left;' +
                     '  font-size: 28px;' +
                     '} ' +
                     '</style>' +
                     '<content></content>';
  </script>
</body>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-styles2.jpg 'Shadow DOM host styles' %}

Adding a red border to our widget doesn't seem like much but there's actually a number of interesting things happening here. For starters, notice that styles applied to the `:host` are inherited by elements within the shadow DOM. So our `<p>` ends up with a `font-size` of 28px.

Also notice that the page is able to set the `text-align` inside the `:host` to center. The `:host` selector has low specificity by design, so it's easier for the document to override it if it needs to. In this case the document style for `.widget` beats out the shadow style for `:host`.

## Styling by :host Type

### Sketch 6: [styling-host-by-host-type](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/06-styling-by-host-type)

Because `:host` is a pseudo selector we can apply it to more than one tag to change the appearance of our component. Let's do another example to demonstrate.

```html
<body>
  <p>My Paragraph</p>
  <div>My Div</div>
  <button>My Button</button>
  
  <!-- Our template -->
  <template class="shadow-template">
    <style>
      p:host {
        color: blue;
      }

      div:host {
        color: green;
      }

      button:host {
        color: red;
      }

      *:host {
        font-size: 24px;
      }
    </style>
    <content select=""></content>
  </template>

  <script>
    // Create a shadow root for each element
    var root1 = document.querySelector('p').createShadowRoot();
    var root2 = document.querySelector('div').createShadowRoot();
    var root3 = document.querySelector('button').createShadowRoot();

    // We'll use the same template for each shadow root
    var template = document.querySelector('.shadow-template');

    // Stamp the template into each shadow root. Notice how
    // the different :host styles affect the appearance
    root1.appendChild(template.content.cloneNode(true));
    root2.appendChild(template.content.cloneNode(true));
    root3.appendChild(template.content.cloneNode(true));
  </script>
</body>
```

{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-styles3.jpg 'Shadow DOM styling by host type' %}

*I've switched to using a [template tag](/blog/2013/03/16/html5-template-tag-introduction/) for this example since it makes working with the Shadow DOM a lot easier.*

As you can see from the example above, we're able to change the look of our component by matching the `:host` selector to a specific tag. We can also match against classes, IDs, attributes, etc. Really any valid CSS will do.

For instance, you could have `.widget-fixed`, `.widget-flex` and `.widget-fluid` `:hosts` if you wanted to build a highly responsive component. Or `.valid` and `.error` `:hosts` for form elements.

By using the `*` selector we're able to create default styles which will apply to any `:host`, in this case setting all components to a `font-size` of 24px. This way you can construct the basic look for your component and then augment it with different `:host` types.

What about theming hosts based on their parent element? Well, there's a selector for that too!

## Theming
### Sketch 7: [theming](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/07-theming)

```html
<body>
  <div class="serious">
    <p class="serious-widget">
      I am super serious, guys.
    </p>
  </div>

  <div class="playful">
    <p class="playful-widget">
      Pretty little clouds...
    </p>
  </div>

  <template class="widget-template">
    <style>
      :host(.serious) {
        width: 200px;
        height: 50px;
        padding: 50px;
        font-family: monospace;
        font-weight: bold;
        font-size: 24px;
        color: black;
        background: tomato;
      }

      :host(.playful) {
        width: 200px;
        height: 50px;
        padding: 50px;
        font-family: cursive;
        font-size: 24px;
        color: white;
        background: deepskyblue;
      }
    </style>
    <content></content>
  </template>
  <script>
    var root1 = document.querySelector('.serious-widget').createShadowRoot();
    var root2 = document.querySelector('.playful-widget').createShadowRoot();
    var template = document.querySelector('.widget-template');
    root1.appendChild(template.content.cloneNode(true));
    root2.appendChild(template.content.cloneNode(true));
  </script>
</body>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-styles5.jpg 'Shadow DOM theming' %}

Using `:host()` syntax we're able to completely change the look of our widget based on the containing element. This is pretty neat! I'm sure you've all used the child selector before, `.parent > .child`, but have you ever wished for a parent selector, `.parent < .child`? Now it's possible, but only with the shadow DOM. I wonder if we'll see this syntax tracked back to normal CSS someday?

## Styling :host States
### Sketch 8: [styling-host-states](https://github.com/robdodson/webcomponents-sketchbook/tree/master/shadow-dom/08-styling-host-states)

One of the best uses of the `:host` tag is for styling states like `:hover` or `:active`. For instance, let's say you want to add a green border to a button when the user rolls over it. Easy!

```html
<body>
  <button>My Button</button>

  <template class="button-template">
    <style>
      *:host {
        font-size: 18px;
        cursor: pointer;
      }
      *:host:hover {
        border: 2px solid green;
      }
    </style>
    <content></content>
  </template>
  <script>
    var host = document.querySelector('button');
    var root = host.createShadowRoot();
    var template = document.querySelector('.button-template');
    root.appendChild(template.content.cloneNode(true));
  </script>
</body>
```
{% img center http://robdodson.s3.amazonaws.com/images/shadow-dom-styles4.jpg 'Shadow DOM host state' %}

Nothing fancy but hopefully it gets your imagination going a bit. What other states do you think you could create?

## Conclusion

There's still a lot more to talk about when it comes to styling the Shadow DOM. Let's take a break for today and pick it up again tomorrow. As always if you have questions [hit me up on twitter](http://twitter.com/rob_dodson) or leave a comment. Thanks!

## [Continue to Shadow DOM: Styles (cont.) &rarr;](/blog/2013/08/29/shadow-dom-styles-cont-dot/)