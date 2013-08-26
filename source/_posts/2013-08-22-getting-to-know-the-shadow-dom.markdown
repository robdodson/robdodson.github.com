---
layout: post
title: "Getting to Know the Shadow DOM"
date: 2013-08-22 12:02
comments: true
published: false
categories: [HTML5, Shadow DOM, Web Components]
---

## Table of Contents

- [Introduction](/blog/2013/08/26/shadow-dom-introduction/)
- [The Basics](#)
- [Styles](#)
- [Styles (continued)](#)
- [Advanced Insertions](#)
- [Events & User Interaction](#)

## Content vs Presentation

light DOM, shadow DOM, composed DOM

## Polymer example

## Shadow Trees https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#concepts
document
|
nodes
|
node/shadow host
|
shadow root

"The content of a shadow host isn’t rendered; the content of the shadow root is rendered instead."

"One rule of thumb, which is possibly being violated here, is that you shouldn’t put content in Shadow DOM. Content must be in the document to be accessible to screen readers, search engines, extensions and so on. Shadow DOM is for all of the semantically meaningless markup you need to create an attractive, usable widget. But the content should stay in the page."

## Encapsulation

The [ownerDocument](http://www.w3.org/TR/domcore/#dom-node-ownerdocument) property of a node in a shadow tree must refers to the document of the shadow host which hosts the shadow tree.

Window object named properties must access the nodes in the root tree.???

## Insertion Points & Distribution

The content element

Matching Criteria

[::content pseudo selector](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#content-pseudo-element)

## Shadow Insertion Points

http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/

"Shadow insertion points" (<shadow>) are similar to normal insertion points (<content>) in that they're placeholders. However, instead of being placeholders for a host's content, they're hosts for other shadow trees. It's Shadow DOM Inception!

## Events

[Event Dispatch targeting](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#event-retargeting) Upon completion of the event dispatch, the Event object's target and currentTarget must be to the highest ancestor's relative target. Since it is possible for a script to hold on to the Event object past the scope of event dispatch, this step is necessary to avoid revealing the nodes in shadow trees.

When an event is dispatched in a shadow tree, its path either crosses the *shadow boundary* or is terminated at the shadow boundary. One exception are the mutation events. The mutation event types must never be dispatched in a shadow tree.

[Events that are always stopped](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#events-that-are-always-stopped)

[Event retargeting](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#event-retargeting)

[Retargeting relatedTarget](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#retargeting-related-target) Note that retargeting affects touch and focus events.

[Event Retargeting Example](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#event-retargeting-example)

## Styles

Style boundaries

CSS variables

[@host Rule](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#host-at-rule)

[Part Pseudo Element](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#part-pseudo-element)

## User Interaction

Accordingly, selections may only exist within one node tree, because they are defined by a single range. The selection, returned by the window.getSelection() method must never return a selection within a shadow tree. ???

## HTML Elements

A subset of HTML elements must behave as inert, or not part of the document tree. This is consistent how the HTML elements would behave in a document fragment. These elements are:

base
link

Does this mean we can't use link tags in Shadow DOM?

[Forms](https://dvcs.w3.org/hg/webcomponents/raw-file/tip/spec/shadow/index.html#html-forms)