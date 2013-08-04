---
layout: post
title: "nth-child is weird"
date: 2013-08-04 10:55
comments: true
categories: [CSS]
---

I ran into a CSS bug today and it brought up an interesting (and important) question: What's the difference between nth-child and nth-of-type?

<!--more-->

## Comparing two things or one

Take a look at this example codepen to get a feel for what we're talking about.

<div data-height="268" data-theme-id="0" data-slug-hash="GzuKH" data-user="robdodson" data-default-tab="html" class='codepen'><pre><code>&lt;div&gt;
  &lt;!-- uncomment to break --&gt;
  &lt;!-- &lt;h2&gt;I broke everything&lt;&#x2F;h2&gt; --&gt;
  &lt;span&gt;This span is green!&lt;&#x2F;span&gt;
  &lt;span&gt;This span is not green :(&lt;&#x2F;span&gt;
  &lt;em&gt;This one isn&#x27;t even a span!&lt;&#x2F;em&gt;
  &lt;span&gt;Also not green.&lt;&#x2F;span&gt;
  &lt;i&gt;Foobar!&lt;&#x2F;i&gt;
&lt;&#x2F;div&gt;</code></pre>
<p>See the Pen <a href='http://codepen.io/robdodson/pen/GzuKH'>%= penName %></a> by Rob Dodson (<a href='http://codepen.io/robdodson'>@robdodson</a>) on <a href='http://codepen.io'>CodePen</a></p>
</div><script async src="http://codepen.io/assets/embed/ei.js"></script>

I'll start by saying that everything I'm about to say has (of course) already been explained by [Chris Coyier on CSS-Tricks.](http://css-tricks.com/the-difference-between-nth-child-and-nth-of-type/)

I'll quote from Chris' post:

{% blockquote %}
Our :nth-child selector, in "Plain English," means select an element *if*:

It is a paragraph element
It is the second child of a parent

Our :nth-of-type selector, in "Plain English," means:

Select the second paragraph child of a parent
{% endblockquote %}

Chris points out that `nth-child` contains more conditions than `nth-of-type` and as a result `nth-of-type` is probably more useful.