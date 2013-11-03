---
layout: post
title: "Image Slider with Web Components"
date: 2013-11-02 16:28
comments: true
categories: 
---

Repetition is the key to building any habit, both good and bad. We do many repetative tasks as front-end developers and key amongst these is writing (or copying and pasting) the same blobs of HTML, over and over, to produce UI elements.

Think about how you currently implement an image slider, it might look something like this:

```
<div id="slider">
  <input checked="" type="radio" name="slider" id="slide1" selected="false">
  <input type="radio" name="slider" id="slide2" selected="false">
  <input type="radio" name="slider" id="slide3" selected="false">
  <input type="radio" name="slider" id="slide4" selected="false">
  <div id="slides">
    <div id="overflow">
      <div class="inner">
        <article>
          <img src="./rock.jpg" alt="an interesting rock">
        </article>
        <article>
          <img src="./grooves.jpg" alt="some neat grooves">
        </article>
        <article>
          <img src="./arch.jpg" alt="a rock arch">
        </article>
        <article>
          <img src="./sunset.jpg" alt="a dramatic sunset">
        </article>
      </div>
    </div>
  </div>
  <label for="slide1"></label>
  <label for="slide2"></label>
  <label for="slide3"></label>
  <label for="slide4"></label>
</div>
```
<small>Responsive Image Slider adapted from http://csscience.com/responsiveslidercss3/</small>

That's a decent chunk of HTML, and we haven't even included the CSS yet! But imagine if we could remove all of that extra cruft and reduce it down to only the important bits. What would that look like?

```
<img-slider>
  <img src="./sunset.jpg" alt="a dramatic sunset">
  <img src="./arch.jpg" alt="a rock arch">
  <img src="./grooves.jpg" alt="some neat grooves">
  <img src="./rock.jpg" alt="an interesting rock">
</img-slider>
```
Not too shabby! We've ditched the boilerplate and the only code that's left is the stuff <em>we</em> care about. But how would that be possible? The markup has to come from somewhere! And we can't just make up our own tags, can we?

## Hidden in the shadows

For years the browser makers have had a sneaky trick up their sleeve which they're just now revealing. I want you to take a look at this `<video>` tag and really think about all the visual goodies you get with just one line of HTML.

<video src="./foo.webm" controls></video>


There's a play button, a scrubber, timecodes and a volume slider. Lots of stuff that you didn't have to write any markup for, it just appeared when you asked for `<video>`. Like magic.

But, like all magic, it's actually a trick! You see, the browser makers needed a way to guarantee that the tags they implemented would always render the same, regardless of any wacky HTML or CSS we might already have on the page. To do this, they created a secret passage way where they could hide their code and keep it out of our hot little hands. They call this secret place: <strong>the Shadow DOM.</strong>

If you happen to be running Google Chrome you can open your Developer Tools and enable the <code>Show Shadow DOM</code> flag. That'll let you inspect the `<video>` element in more detail.

<img src="./show-shadow-dom.jpg" alt="Enable Show Shadow DOM">
<img src="./video-shadow-dom.jpg" alt="Inspecting Show Shadow DOM">

Inside you'll find that there's actually a ton of HTML all hidden away. Poke around long enough and you'll discover the aforementioned play button, volume slider, and various other elements.

Now, think back to our image slider. What if we <em>all</em> had access to the shadow DOM and the ability to declare our own tags like `<video>`? Then we could make our `<img-slider>` a reality! And as it turns out, that power is in our hands today, and the technologies that enable it are collectively called <strong>Web Components.</strong>

## Web Components?

Web Components are a collection of new standards which are working their way through the W3C and landing in browsers as we speak. In a nutshell, they allow us to bundle markup and styles into custom HTML elements. What's truly amazing about these new elements is that they fully encapsulate all of their HTML and CSS. That means the styles that you write always render as you intended, and your HTML is safe from the prying eyes of external JavaScript.

Let's look at how we can turn our `img-slider` into a real tag, using Web Components.

## Templates

Every good construction project has to start with a blueprint, and with Web Components that blueprint comes from the new `<template>` tag. The template tag allows you to store some markup on the page which you can later clone and reuse. If you've worked with libraries like mustache or handlebars before, then the `<template>` tag should feel familiar.

```
<template>
  <h1>Hello there!</h1>
  <p>This content is top secret :)</p>
</template>
```
Everything inside a template is considered inert by the browser. This means tags with external sources—`<img>`, `<audio>`, `<video>`, etc.—do not make http requests and `<script>` tags do not execute. It also means that nothing from within the template is rendered on the page until we activate it using JavaScript.

So the first step in creating our `<img-slider>` is to put all of its HTML and CSS into a `<template>`.

<p data-height="268" data-theme-id="0" data-slug-hash="EeLav" data-user="robdodson" data-default-tab="html" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/EeLav'>CSS3 Slider Template</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
<small>http://codepen.io/robdodson/pen/EeLav</small>

Once we've done this, we're ready to move it into the shadow DOM.

## Shadow DOM

To really make sure that our HTML and CSS doesn't adversly affect the consumer we sometimes resort to iframes. They do the trick, but you wouldn't want to build your entire application in 'em.

Shadow DOM gives us the best features of iframes, style and markup encapsulation, without nearly as much bloat.

To create shadow DOM, select an element and call its `createShadowRoot` method. This will return a document fragment which you can then fill with content.

```
<div class="container"></div>

&lt;script&gt;
  var host = document.querySelector('.container');
  var root = host.createShadowRoot();
  root.innerHTML = '<p>How <em>you</em> doin?</p>'
&lt;/script&gt;
```

### Shadow Host

In shadow DOM parlance, the element that you call `createShadowRoot` on is known as the <strong>Shadow Host.</strong> It's the only piece visible to the user, and it's where you would ask the user to supply your element with content.

If you think about our `<video>` tag from before, the `<video>` element itself is the shadow host, and the contents are the `<source>` tags you nest inside of it.

```
<video>
  <source src="trailer.mp4" type="video/mp4">
  <source src="trailer.webm" type="video/webm">
  <source src="trailer.ogv" type="video/ogg">
</video>
```

### Shadow Root

The document fragment returned by `createShadowRoot` is known as the <strong>Shadow Root.</strong> The shadow root, and its descendants, are hidden from the user, but they're what the browser will actually render when it sees our tag. In the `<video>` example, the play button, scrubber, timecode, etc. are all descendants of the shadow root.


### Shadow Boundary

Any HTML and CSS inside of the shadow root is protected from the parent document by an invisible barrier called the <strong>Shadow Boundary.</strong> The shadow boundary prevents CSS in the parent document from bleeding into the shadow DOM and it also prevents external JavaScript from traversing into the shadow root.

Translation: Let's say you have a style tag in the shadow DOM that specifies all h3's should have a `color` of red. Meanwhile, in the parent document, you have a style that specifies h3's should have a `color` of blue. In this instance, h3's appearing within the shadow DOM will be red, and h3's outside of the shadow DOM will be blue. The two styles will happily ignore each other thanks to our friend the shadow boundary.

And if, at some point, the parent document goes looking for h3's with `$('h3')`, the shadow boundary will prevent any exploration into the shadow root and the selection will only return h3's that are external to the shadow DOM.

This level of privacy is something that we've dreamed about and worked around for years, and now, suddenly, it's here. To say that it will change the way we build web applications is truly understatment.

To get our `img-slider` into the shadow DOM we'll need to create a shadow host and populate it with the contents of our template.

<p data-height="268" data-theme-id="0" data-slug-hash="GusaF" data-user="robdodson" data-default-tab="js" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/GusaF'>CSS3 Slider Shadow DOM</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
<small>http://codepen.io/robdodson/pen/GusaF</small>

We can access the internals of the template with the `content` keyword. Calling `cloneNode(true)` returns a deep copy of those internals which we then append to the shadow root.

### Insertion Points

At this point our `img-slider` is inside the shadow DOM but we still need to add the actual images. We'd like the images to come from the user, so we'll have them over from the shadow host.

To pull items into the shadow DOM we use the new `<content>` tag. The `<content>` tag uses CSS selectors to cherry-pick elements from the shadow host and project them into the shadow DOM. This is known as an <strong>insertion point.</strong>

We'll make it easy on ourselves and assume that the slider always has 4 images, that way we can select with `nth-of-type`.

```
<template>
  ...
  <div class="inner">
    <article>
      <content select="img:nth-of-type(1)"></content>
    </article>
    <article>
      <content select="img:nth-of-type(2)"></content>
    </article>
    <article>
      <content select="img:nth-of-type(3)"></content>
    </article>
    <article>
      <content select="img:nth-of-type(4)"></content>
    </article>
  </div>
</template>
```

Now we can populate our `img-slider`.

```
<div class="img-slider">
  <img src="./rock.jpg" alt="an interesting rock">
  <img src="./grooves.jpg" alt="some neat grooves">
  <img src="./arch.jpg" alt="a rock arch">
  <img src="./sunset.jpg" alt="a dramatic sunset">
</div>
```

This is really cool, but we can actually take things a step further and turn our `img-slider` into its very own HTML element.

## Custom Elements

Creating your own HTML element might sound intimidating but it's actually quite easy. In Web Components speak, this new element is a <strong>Custom Element</strong>, and the only two requirements are that its name must contain a dash, and its prototype must extend `HTMLElement`.

Let's take a look at how that might work.

```
<template>
  <!-- Full of image slider awesomeness -->
</template>

&lt;script&gt;
  // Grab our template full of slider markup and styles
  var tmpl = document.querySelector('template');

  // Create a prototype for a new element that extends HTMLElement
  var ImgSliderProto = Object.create(HTMLElement.prototype);

  // Setup our Shadow DOM and clone the template
  ImgSliderProto.createdCallback = function() {
    var root = this.createShadowRoot();
    root.appendChild(tmpl.content.cloneNode(true));
  };

  // Register our new element
  var ImgSlider = document.register('img-slider', {
    prototype: ImgSliderProto
  });
&lt;/script&gt;
```

The `Object.create` method returns a new prototype which extends HTMLElement. When the parser sees our tag in the document it will check to see if there's a method named `createdCallback` on its prototype. If it finds the method it will run it immediately. This is a good place to do setup work, so we create some Shadow DOM and clone our template into it.

We pass the tag name and prototype to a new method on the `document` object called `register()` and after that we're good to go!

Now that our element is registered there are a few different ways to use it. The first, and most straightforward, is to just use the new tag `<img-slider></img-slider>` in our HTML. We can also call `document.createElement("img-slider")` or we can use the constructor that was returned by `document.register` and stored in the `ImgSlider` variable. It's up to you which style you prefer.

## Support

Support for the various standards that makeup Web Components is currently spotty, though improving all the time. This table illustrates where we're presently at.

<img src="./support.png" alt="Web Components Support Table">

But don't let that discourage you from using them! The smarties at Mozilla and Google have been hard at work building polyfill libraries which sneak support for Web Components into all modern browsers! This means you can start playing with these technologies today and give feedback to the folks writing the specs so we don't end up with stinky, hard to use syntax. Let's look at how we could rewrite our `img-slider` using Google's polyfill library, Polymer.

## Polymer to the Rescue!

Polymer adds a new tag to the browser, `<polymer-element>`, which automagically turns templates into Shadow DOM and registers Custom Elements for us. All we need to do is to tell Polymer what name to use for the tag and to make sure we include our template markup.

```
<polymer-element name="img-slider" noscript>
  <template>
    <!-- Full of image slider awesomeness -->
  </template>
</polymer-element>
```

You can checkout the working `img-slider` as a Polymer element in this pen.

// http://codepen.io/robdodson/pen/blfGh

Polymer actually does *way* more than just turning templates into Shadow DOM and Custom Elements, but we'll have to save that for another post :)

## Issues

Any new standard can be contriversial and in the case of Web Components it seems that they are especially polarizing. 

### OMG it's XML!!!

I think the thing that probably scares most developers when they first see Custom Elements is the notion that it will turn the document into one big pile of XML where everything on the page has some bespoke tag name and in this fashion we'll make the web pretty much unreadable. I think that's a valid argument so I decided to kick the bees' nest and [bring it up on the Polymer mailing list.](https://groups.google.com/forum/#!searchin/polymer-dev/xml/polymer-dev/lzvaDViB_Ow/VtbeIqX0Ap0J)

The back and forth discussion is pretty interesting but I think the general consensus is that we're just going to have to experiment to see what works and what doesn't.

### SEO

At this moment it's unclear (at least to me) how well crawlers support Custom Elements and Shadow DOM. The Polymer FAQ states:

>search engines have been dealing with heavy AJAX based application for some time now. Moving away from JS and being more declarative is a good thing and will generally make things better.

Others have pointed out that this explanation falls short and [there's a pretty interesting discussion on the topic over in the Polymer mailing list.](https://groups.google.com/forum/#!searchin/polymer-dev/seo/polymer-dev/TMBtl3_bDT0/VwR9-t6V--AJ)

### Accessibility

Obviously when you're hiding markup in secret Shadow DOM sandboxes the issue of accessibility becomes pretty important. Steve Faulkner took a look at accessibility in Shadow DOM and seemed to be fairly satisfied with what he found.

> Results from initial testing indicate that inclusion of ARIA roles, states and properties in content wholly inside the Shadow DOM works fine. The accessibility information is exposed correctly via the accessibility API. Screen readers can access content in the Shadow DOM without issue.

[The full post is available here.](http://blog.paciellogroup.com/2012/07/notes-on-web-components-aria/)

### Style and Script tags? Um, no thanks.

Unfortunately it's a limitation of the HTML Import spec (which we haven't touched on) that external scripts and stylesheets are not currently supported in Web Components. In Polymer you can use @import to pull in external CSS but in native Web Components you're out of luck. While this isn't great, we have to keep in mind that the scripts and styles that we're talking about are relevant only to the component whereas we've previously been trained to favor external files because they often affect our entire application. So is it such a bad thing to put a <style> tag inside of an element, if all of those styles are scoped to that particular component?











