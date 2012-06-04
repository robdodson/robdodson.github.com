---
layout: post
title: "CSS3 Transitions with GFX"
date: 2012-05-22 08:06
comments: true
categories: [Chain, CSS3, Animation, GFX]
---

CSS3 is a rather verbose tool especially when it comes to transitions and animations. I want to see if there's a way to clean a lot of that up with either a SASS mixin or a jQuery library like [gfx.](http://maccman.github.com/gfx/)

We'll try to do something simple at first to recreate an animation like the one we have below.

<small>Rollover the grey area to activate it.</small>
<!-- CSS Styles: -->
<div>
  <style type="text/css">
    .container {
      width: 300px;
      height: 300px;
      position: relative;
      background-color: #CCC;
    }

    #example1 .container:hover .widget {
      background-color: red;
      left: 200px;
      top: 200px;
    }

    #example1 .widget {
      background-color: black;
      left: 10px;
      position: absolute;
      top: 10px;
      width: 20px;
      height: 20px;
      -webkit-transition: background-color 1s linear, left 1s, top 2s;
      -moz-transition: background-color 1s linear, left 1s, top 2s;
      -ms-transition: background-color 1s linear, left 1s, top 2s;
      -o-transition: background-color 1s linear, left 1s, top 2s;
      transition: background-color 1s linear, left 1s, top 2s;
    }
  </style>
</div>

<div id="example1">
  <div class="container">
      <div class="widget"></div>
  </div>​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​
</div>

The code to produce the above animation looks something like this:

``` css
.container {
  width: 300px;
  height: 300px;
  position: relative;
  background-color: #CCC;
}

.container:hover .widget {
  background-color: red;
  left: 200px;
  top: 200px;
}

.widget {
  background-color: black;
  left: 10px;
  position: absolute;
  top: 10px;
  width: 20px;
  height: 20px;
  -webkit-transition: background-color 1s linear, left 1s, top 2s;
  -moz-transition: background-color 1s linear, left 1s, top 2s;
  -ms-transition: background-color 1s linear, left 1s, top 2s;
  -o-transition: background-color 1s linear, left 1s, top 2s;
  transition: background-color 1s linear, left 1s, top 2s;
}
```
We definitely don't want all that logic in our CSS. Ideally we could trigger these kinds of things from JavaScript, which is where Gfx comes in.

Using Gfx we can get similar but not exactly the same result with the following snippet:

``` js
$("#widget")
    .gfx({ translateX: '200px' }, { duration: '1000', queue: false })
    .gfx({ translateX: '200px', translateY: '200px' }, { duration: '2000' });
```

After tinkering with Gfx for what seems like hours at this point I haven't figured out a way to pass two different times to simultaneous animations. From one animation to the next you have to specify the end points from the previous animations or else the styles will revert. That's why I'm passing `translateX: '200px'` in both places...although I'm not 100% if that's what I should be doing. I emailed the author of Gfx so we'll see what he says :)

*Update: Alex responded:*

>atm gfx is -webkit-transition: all - perhaps that's something that should be added. 

I've looked at other libraries including [jQuery Transit](http://ricostacruz.com/jquery.transit/) and [Move.js](http://visionmedia.github.com/move.js/), and they all seem to be set to the 'all' mode. If anyone knows of a good library that can handle simultaneous animations I would love to hear about it. I think otherwise you would just need to create two animation objects and run them at the same time... but I haven't tested that theory yet.

- Time: 8:07 am
- Mood: Sedated, Upset-Stomach, Laggy
- Sleep: 5.5
- Hunger: 0
- Coffee: 0
