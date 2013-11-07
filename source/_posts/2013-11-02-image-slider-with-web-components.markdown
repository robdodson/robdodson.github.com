---
layout: post
title: "A Modular Future with Web Components"
date: 2013-11-02 16:28
comments: true
published: false
categories: 
---

Recently I was working with a client to train their internal teams on how to build web applications. During this process it occurred to me that the way we presently architect the front-end is very strange and even a bit broken. In many instances you’re either copying huge chunks of HTML out of some doc and then pasting that into your app (Bootstrap, Foundation, etc.), or you’re sprinkling the page with jQuery plugins which have to be configured using JavaScript . It puts us in the rather unfortunate position of having to choose between bloated HTML or mysterious HTML, and often we choose **both.**

In an ideal scenario, the HTML language would be expressive enough to create complex UI widgets and also extensible so that we, the developers, could fill in any gaps with our own tags. Today, this is finally possible through a new set of standards called **Web Components.**

## Web Components?

Web Components are a collection of standards which are working their way through the W3C and landing in browsers as we speak. In a nutshell, they allow us to bundle markup and styles into custom HTML elements. What's truly amazing about these new elements is that they fully encapsulate all of their HTML and CSS. That means the styles that you write always render as you intended, and your HTML is safe from the prying eyes of external JavaScript.

If you want to play with native Web Components I'd recommend using [Chrome Canary](https://www.google.com/intl/en/chrome/browser/canary.html), since it has the best support. Be sure to enable the following settings in `chrome://flags`.

```
Experimental JavaScript
Experimental Web Platform Features
HTML Imports
```

## Le Practical Example

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

<p data-height="460" data-theme-id="0" data-slug-hash="rCGvJ" data-user="robdodson" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/rCGvJ'>CSS3 Slider</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

<small>
  Responsive Image Slider adapted from <a href="http://csscience.com/responsiveslidercss3/">CSScience</a>. 
  Images courtesy <a href="http://www.flickr.com/photos/eliya">Eliya Selhub</a>
</small>

That's a decent chunk of HTML, and we haven't even included the CSS yet! But imagine if we could remove all of that extra cruft and reduce it down to only the important bits. What would that look like?

```
<img-slider>
  <img src="./sunset.jpg" alt="a dramatic sunset">
  <img src="./arch.jpg" alt="a rock arch">
  <img src="./grooves.jpg" alt="some neat grooves">
  <img src="./rock.jpg" alt="an interesting rock">
</img-slider>
```
Not too shabby! We've ditched the boilerplate and the only code that's left is the stuff we care about. This is the kind of thing that Web Components will allow us to do. But before I delve into the specifics I'd like to tell you another story.

## Hidden in the shadows

For years the browser makers have had a sneaky trick hidden up their sleeves. Take a look at this <code>&lt;video&gt;</code> tag and really think about all the visual goodies you get with just one line of HTML.

<video src="./foo.webm" controls></video>

There's a play button, a scrubber, timecodes and a volume slider. Lots of stuff that you didn't have to write any markup for, it just appeared when you asked for <code>&lt;video&gt;</code>.

But what you’re actually seeing is an illusion. The browser makers needed a way to guarantee that the tags they implemented would always render the same, regardless of any wacky HTML, CSS or JavaScript we might already have on the page. To do this, they created a secret passage way where they could hide their code and keep it out of our hot little hands. They called this secret place: <strong>the Shadow DOM.</strong>

If you happen to be running Google Chrome you can open your Developer Tools and enable the <code>Show Shadow DOM</code> flag. That'll let you inspect the <code>&lt;video&gt;</code> element in more detail.

<img src="./show-shadow-dom.jpg" alt="Enable Show Shadow DOM">
<img src="./video-shadow-dom.jpg" alt="Inspecting Show Shadow DOM">

Inside you'll find that there's a ton of HTML all hidden away. Poke around long enough and you'll discover the aforementioned play button, volume slider, and various other elements.

Now, think back to our image slider. What if we <em>all</em> had access to the shadow DOM and the ability to declare our own tags like <code>&lt;video&gt;</code>?  Then we could actually implement and use our custom <code>&lt;img-slider&gt;</code> tag.

Let’s take a look at how to make this happen, using the first pillar of Web Components, the template.

## Templates

Every good construction project has to start with a blueprint, and with Web Components that blueprint comes from the new <code>&lt;template&gt;</code> tag. The template tag allows you to store some markup on the page which you can later clone and reuse. If you've worked with libraries like mustache or handlebars before, then the <code>&lt;template&gt;</code> tag should feel familiar.

```
<template>
  <h1>Hello there!</h1>
  <p>This content is top secret :)</p>
</template>
```
Everything inside a template is considered inert by the browser. This means tags with external sources—<code>&lt;img&gt;</code>, <code>&lt;audio&gt;</code>, <code>&lt;video&gt;</code>, etc.—do not make http requests and <code>&lt;script&gt;</code> tags do not execute. It also means that nothing from within the template is rendered on the page until we activate it using JavaScript.

So the first step in creating our <code>&lt;img-slider&gt;</code> is to put all of its HTML and CSS into a <code>&lt;template&gt;</code>.

<p data-height="268" data-theme-id="0" data-slug-hash="EeLav" data-user="robdodson" data-default-tab="html" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/EeLav'>CSS3 Slider Template</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>
<small>http://codepen.io/robdodson/pen/EeLav</small>

Once we've done this, we're ready to move it into the shadow DOM.

## Shadow DOM

To really make sure that our HTML and CSS doesn't adversely affect the consumer we sometimes resort to iframes. They do the trick, but you wouldn't want to build your entire application in 'em.

Shadow DOM gives us the best features of iframes, style and markup encapsulation, without nearly as much bloat.

To create shadow DOM, select an element and call its <code>createShadowRoot</code> method (or <code>webkitCreateShadowRoot</code> if you're using Chrome 30 or Safari). This will return a document fragment which you can then fill with content.

```
<div class="container"></div>

&lt;script&gt;
  var host = document.querySelector('.container');
  var root = host.createShadowRoot();
  root.innerHTML = '<p>How <em>you</em> doin?</p>'
&lt;/script&gt;
```

### Shadow Host

In shadow DOM parlance, the element that you call <code>createShadowRoot</code> on is known as the <strong>Shadow Host.</strong> It's the only piece visible to the user, and it's where you would ask the user to supply your element with content.

If you think about our <code>&lt;video&gt;</code> tag from before, the <code>&lt;video&gt;</code> element itself is the shadow host, and the contents are the <code>&lt;source&gt;</code> tags you nest inside of it.

```
<video>
  <source src="trailer.mp4" type="video/mp4">
  <source src="trailer.webm" type="video/webm">
  <source src="trailer.ogv" type="video/ogg">
</video>
```

### Shadow Root

The document fragment returned by <code>createShadowRoot</code> is known as the <strong>Shadow Root.</strong> The shadow root, and its descendants, are hidden from the user, but they're what the browser will actually render when it sees our tag.

In the <code>&lt;video&gt;</code> example, the play button, scrubber, timecode, etc. are all descendants of the shadow root. They show up on the screen but their markup is not visible to the user.

### Shadow Boundary

Any HTML and CSS inside of the shadow root is protected from the parent document by an invisible barrier called the <strong>Shadow Boundary.</strong> The shadow boundary prevents CSS in the parent document from bleeding into the shadow DOM, and it also prevents external JavaScript from traversing into the shadow root.

Translation: Let's say you have a style tag in the shadow DOM that specifies all h3's should have a <code>color</code> of red. Meanwhile, in the parent document, you have a style that specifies h3's should have a <code>color</code> of blue. In this instance, h3's appearing within the shadow DOM will be red, and h3's outside of the shadow DOM will be blue. The two styles will happily ignore each other thanks to our friend, the shadow boundary.

And if, at some point, the parent document goes looking for h3's with <code>$('h3')</code>, the shadow boundary will prevent any exploration into the shadow root and the selection will only return h3's that are external to the shadow DOM.

This level of privacy is something that we've dreamed about and worked around for years. To say that it will change the way we build web applications is a total understatement.

## Shadowy Sliders

To get our <code>img-slider</code> into the shadow DOM we'll need to create a shadow host and populate it with the contents of our template.

```
<template>
  <!-- Full of slider awesomeness -->
</template>

<div class="img-slider"></div>

&lt;script&gt;
  // Add the template to the Shadow DOM
  var tmpl = document.querySelector('template');
  var host = document.querySelector('.img-slider');
  var root = host.createShadowRoot();
  // host.webkitCreateShadowRoot() in Safari and Chrome 30
  root.appendChild(tmpl.content.cloneNode(true));
&lt;/script&gt;
```

In this instance we've created a <code>div</code> and given it the class <code>img-slider</code> so it can act as our shadow host.

We select the template and access the internals with the <code>content</code> keyword. Calling <code>cloneNode(true)</code> returns a deep copy of those internals which we then append to the shadow root.

If you're using Chrome or Safari you can actually see this working in the following pen.

<p data-height="460" data-theme-id="0" data-slug-hash="GusaF" data-user="robdodson" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/GusaF'>CSS3 Slider Shadow DOM</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

### Insertion Points

At this point our <code>img-slider</code> is inside the shadow DOM but the image paths are hard coded. Just like the <code>&lt;source&gt;</code> tags nested inside of <code>&lt;video&gt;</code>, we'd like the images to come from the user, so we'll have to invite them over from the shadow host.

To pull items into the shadow DOM we use the new <code>&lt;content&gt;</code> tag. The <code>&lt;content&gt;</code> tag uses CSS selectors to cherry-pick elements from the shadow host and project them into the shadow DOM. These projections are known as <strong>insertion points.</strong>

We'll make it easy on ourselves and assume that the slider always has four images, that way we can create four insertion points using <code>nth-of-type</code>.

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

Now we can populate our <code>img-slider</code>.

```
<div class="img-slider">
  <img src="./rock.jpg" alt="an interesting rock">
  <img src="./grooves.jpg" alt="some neat grooves">
  <img src="./arch.jpg" alt="a rock arch">
  <img src="./sunset.jpg" alt="a dramatic sunset">
</div>
```

This is really cool! We've cut the amount of markup that the user sees way down. But why stop here? We can take things a step further and make this more semantic by turning our <code>img-slider</code> into its very own HTML element.

## Custom Elements

Creating your own HTML element might sound intimidating but it's actually quite easy. In Web Components speak, this new element is a <strong>Custom Element</strong>, and the only two requirements are that its name must contain a dash, and its prototype must extend <code>HTMLElement</code>.

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

The <code>Object.create</code> method returns a new prototype which extends <code>HTMLElement</code>. When the parser finds our tag in the document it will check to see if it has a method named <code>createdCallback</code>. If it finds this method it will run it immediately. This is a good place to do setup work, so we create some Shadow DOM and clone our template into it.

We pass the tag name and prototype to a new method on the <code>document</code>, called <code>register</code>, and after that we're ready to go.

Now that our element is registered there are a few different ways to use it. The first, and most straightforward, is to just use the <code>&lt;img-slider&gt;</code> tag somewhere in our HTML. But we can also call <code>document.createElement("img-slider")</code> or we can use the constructor that was returned by <code>document.register</code> and stored in the <code>ImgSlider</code> variable. It's up to you which style you prefer.

## Support

Support for the various standards that makeup Web Components is currently spotty, though improving all the time. This table illustrates where we're presently at.

<img src="./support.png" alt="Web Components Support Table">

But don't let the table discourage you from using them! The smarties at Mozilla and Google have been hard at work building polyfill libraries which sneak support for Web Components into all modern browsers! This means you can start playing with these technologies <em>today</em> and give feedback to the folks writing the specs. That feedback is important so we don't end up with stinky, hard to use syntax.

Let's look at how we could rewrite our <code>img-slider</code> using Google's polyfill library, <a href="http://www.polymer-project.org/">Polymer.</a>

## Polymer to the Rescue!

Polymer adds a new tag to the browser, <code>&lt;polymer-element&gt;</code>, which automagically turns templates into shadow DOM and registers custom elements for us. All we need to do is to tell Polymer what name to use for the tag and to make sure we include our template markup.

<p data-height="460" data-theme-id="0" data-slug-hash="blfGh" data-user="robdodson" data-default-tab="result" class='codepen'>See the Pen <a href='http://codepen.io/robdodson/pen/blfGh'>Polymer Slider</a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
<script async src="//codepen.io/assets/embed/ei.js"></script>

<small>http://codepen.io/robdodson/pen/blfGh</small>

I find it's often easier to create elements using Polymer because of all the niceties built into the library. This includes two-way binding between elements and models, automatic node finding and support for other new standards like Web Animations and Pointer Events. Also, the developers on the <a href="https://groups.google.com/forum/#!forum/polymer-dev">polymer-dev mailing list</a> are extremely active and helpful, which is great when you're first learning the ropes.

This is just a tiny example of what Polymer can do, so be sure to <a href="http://www.polymer-project.org/">visit its project page</a> and also checkout Mozilla's alternative, <a href="http://www.x-tags.org/">X-Tags.</a>

## Issues

Any new standard can be controversial and in the case of Web Components it seems that they are especially polarizing. Before we wrap up, I want to open up for discussion some of the feedback I've heard over the past few months and give my take on it.

### OMG it's XML!!!

I think the thing that probably scares most developers when they first see Custom Elements is the notion that it will turn the document into one big pile of XML, where everything on the page has some bespoke tag name and, in this fashion, we'll make the web pretty much unreadable. I think that's a valid argument so I decided to kick the bees' nest and <a href="https://groups.google.com/forum/#!searchin/polymer-dev/xml/polymer-dev/lzvaDViB_Ow/VtbeIqX0Ap0J">bring it up on the Polymer mailing list.</a>

The back and forth discussion is pretty interesting but I think the general consensus is that we're just going to have to experiment to see what works and what doesn't. Is it better, and more semantic, to see a tag name like <code>&lt;img-slider&gt;</code> or is our present "div soup" the only way it should be?

### SEO

At this moment it's unclear (at least to me) how well crawlers support Custom Elements and Shadow DOM. <a href="http://www.polymer-project.org/faq.html#seo">The Polymer FAQ states</a>:

> Search engines have been dealing with heavy AJAX based application for some time now. Moving away from JS and being more declarative is a good thing and will generally make things better.

Others have pointed out that this explanation falls short and there's <a href="https://groups.google.com/forum/#!searchin/polymer-dev/seo/polymer-dev/TMBtl3_bDT0/VwR9-t6V--AJ">a discussion on the topic over in the Polymer mailing list.</a>

This will need to be worked out before Web Components receive wide adoption but I'm optimistic they'll get it right, primarily because Google is really pushing for it and it wouldn't make sense to implement something that stymied their search engine.

### Accessibility

Obviously when you're hiding markup in secret shadow DOM sandboxes the issue of accessibility becomes pretty important. Steve Faulkner took a look at accessibility in shadow DOM and seemed to be satisfied with what he found.

> Results from initial testing indicate that inclusion of ARIA roles, states and properties in content wholly inside the Shadow DOM works fine. The accessibility information is exposed correctly via the accessibility API. Screen readers can access content in the Shadow DOM without issue.

<a href="http://blog.paciellogroup.com/2012/07/notes-on-web-components-aria/">The full post is available here.</a>

Surely there will be bumps along the way but that sounds like a pretty great start!

### Style and Script tags? Um, no thanks.

Unfortunately it's a limitation of the HTML Import spec (which we haven't touched on) that external scripts and stylesheets are not currently supported in Web Components. In Polymer you can use <code>@import</code> to pull in external CSS but in native Web Components you're out of luck.

Keep in mind that the scripts and styles we're talking about are relevant only to a component, whereas we've previously been trained to favor external files because they often affect our entire application. So is it such a bad thing to put a <code>&lt;style&gt;</code> tag inside of an element, if all of those styles are scoped just to that one entity? Personally I think it's OK, but the option of external scripts would be very nice to have.

## Now it's your turn

It's up to us to figure out where these standards should go and what best practices will guide them. Give <a href="http://www.polymer-project.org/">Polymer</a> a shot, and also look at Mozilla's alternative to Polymer, <a href="http://www.x-tags.org/">X-tags</a> (which has support all the way down to Internet Explorer 9).

Also, make sure you reach out to <a href="https://groups.google.com/forum/#!forum/polymer-dev">the developers at Google</a> and <a href="https://bugzilla.mozilla.org/show_bug.cgi?id=889230">Mozilla</a> who are driving the bus on these standards. It'll take our feedback to properly mold these tools into something we all want to use.

While there are still some rough edges, I think Web Components will eventually usher in a new style of application development, something more akin to snapping together Legos and less like our current approach, which is often plagued by excess boilerplate. I'm pretty excited by where all of this is heading, and I look forward to what the future might hold.