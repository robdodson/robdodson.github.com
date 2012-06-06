---
layout: post
title: "Building a Countdown Timer with Socket.io"
date: 2012-06-05 08:07
comments: true
categories: [Chain, Node, Socket.io, Express]
---

[Click here for part 2.](http://robdodson.me/blog/2012/06/05/building-a-countdown-timer-with-socket-dot-io-pt-2/)

Yesterday I put together a very simple Node/Socket.io application and showed how to deploy it to Heroku. Today I'm going to keep going with that app to see if I can get the functionality that I want. The app is a basic stopwatch so that shouldn't be too hard. If you want to catch up [checkout yesterday's article](http://robdodson.me/blog/2012/06/04/deploying-your-first-node-dot-js-and-socket-dot-io-app-to-heroku/) which explains setting everything up.

### Countdown

Just to get the ball rolling I'm going to write a little code in my `app.js` file right at the bottom to setup a very crude counter.

``` js app.js  
var countdown = 1000;
setInterval(function() {
  countdown--;
  io.sockets.emit('timer', { countdown: countdown });
}, 1000);

io.sockets.on('connection', function (socket) {
  socket.on('reset', function (data) {
    countdown = 1000;
    io.sockets.emit('timer', { countdown: countdown });
  });
});
```

Elsewhere in my client-side js I'm going to listen for the `timer` event and update my DOM elements.

``` js main.js
var socket = io.connect(window.location.hostname);

socket.on('timer', function (data) {
    $('#counter').html(data.countdown);
});

$('#reset').click(function() {
    socket.emit('reset');
});
```

Every second we'll decrement our countdown variable and broadcast its new value. If a client sends us a `reset` event we'll restart the timer and immediately broadcast the update to anyone connected. I noticed that since I'm using `xhr-polling` it can sometimes take a while for the timer to show up in my browser so keep that in mind.

While this implementation isn't pretty it does get us a little bit further down the road. Unfortunately I've been tripped up by a bunch of Node's module issues so I have to cut tonight's post short :\

Hopefully better luck tomorrow. - Rob

You should follow me on Twitter [here.](http://twitter.com/rob_dodson)

- Time: 8:08 am
- Mood: Sedate, Sleepy
- Sleep: 5
- Hunger: 4
- Coffee: 0
