---
layout: post
title: "Managing Your Backbone Views with the State Pattern"
date: 2012-06-02 18:22
comments: true
categories: [Chain, JavaScript, Design Patterns, State Pattern, Backbone, Backbone Views]
---

[Yesterday I wrote a post to illustrate the concepts behind the State pattern](http://robdodson.me/blog/2012/06/02/take-control-of-your-app-with-the-javascript-state-patten/) (one of my all time favorite tools). If you're new to this pattern and haven't read my previous post I suggest you start there and read this one after you've had a chance to play around with the ideas.

<!--more-->

I wanted to write about this pattern in the context of a Backbone app because I think there's a lot of value in exploring different ways to manage our Views and Models. Here's an example Video Player in which several State objects inherit from a common ancestor, `BaseState`. I chose to make `BaseState` extend `Backbone.Model` because it seemed like the best fit for this kind of thing, although typically when I implement this pattern in other languages I just use generic Objects. I would have done that here but Backbone's `extend` functionality makes the code so much cleaner.

``` js
var BaseState = Backbone.Model.extend({
  initialize: function(owner) {
    this.owner = owner;
  },
  enter: function() {
    // Implement me in your state objects
  },
  execute: function() {
    // Implement me in your state objects
  },
  play: function() {
    console.log('changing to the playing state...');
    this.owner.changeState(this.owner.states.playing);
  },
  stop: function() {
    console.log('changing to the stopping state...')
    this.owner.changeState(this.owner.states.stopping);
  },
  pause: function() {
    console.log('changing to the pausing state...')
    this.owner.changeState(this.owner.states.pausing);
  },
  exit: function() {
    // Implement me in your state objects
  }
});

var PlayingState = BaseState.extend({
  execute: function() {
    console.log('playing the video!');
  },
  play: function() {
    console.log('already playing!');
  }
});

var StoppingState = BaseState.extend({
  enter: function() {
    console.log('we have entered the stopping state...');
  },
  execute: function() {
    console.log('stopping the video...');
  },
  stop: function() {
    console.log('already stopped!');
  }
});

var PausingState = BaseState.extend({
  execute: function() {
    console.log('pausing the video.');
  },
  pause: function() {
    console.log('already paused!');
  }
});
``` 

Every state inherits from `BaseState` which means they all create a reference to an `owner` object in their `initialize` methods. The `owner` is going to be our actual `VideoPlayer` object. Rather than having `videoPlayer.play` lead to some big weird conditional:

``` js
play: function() {
  if (this.status == 'playing') {
      return;
  } else if (this.status == 'stopped') {
      // play the video
  } else if (this.status == 'paused') {
      // unpause and play
  }
  else if ...
}
```

we're instead going to delegate the work to our State objects. They'll handle switching from one State to the next and so long as we provide the exact same public API methods in each, they should be interchangeable. Inheriting from the same BaseState ensures that all of the States have the same public methods and each State can choose how or if it wants to override them. In our example the `StoppingState` overrides the `enter` method to display some text as we're transitioning into this state. Again, you can override some or all of the methods, `enter` and `exit` are great for building up/tearing down anything that our state might need and `execute` is where the main work of our state should happen.

Let's take a look at the `VideoPlayer` which is a `Backbone.View` that will leverage our State objects:

``` js
var VideoPlayer = Backbone.View.extend({
  initialize: function() {
    this.states = {};
    this.states.playing = new PlayingState(this);
    this.states.stopping = new StoppingState(this);
    this.states.pausing = new PausingState(this);
    this.changeState(this.states.pausing);
  },
  play: function() {
    this.state.play();
  },
  stop: function() {
    this.state.stop();
  },
  pause: function() {
    this.state.pause();
  },
  changeState: function(state) {
    // Make sure the current state wasn't passed in
    if (this.state !== state) {
      // Make sure the current state exists before
      // calling exit() on it
      if (this.state) {
        this.state.exit();
      }
      this.state = state;
      this.state.enter();
      this.state.execute();
    }
  }
});
```
You'll notice that `VideoPlayer` instantiates its own State objects when it is first created. Then whenever we call one of its public methods the call is delegated to whichever state object is currently residing in `this.state`.

Let's instantiate a new VideoPlayer to see it in action:

``` js
var videoPlayer = new VideoPlayer();

=> 'pausing the video.'


videoPlayer.play();

=> 'changing to the playing state...'
=> 'playing the video!'

videoPlayer.play();

=> 'already playing!' 


videoPlayer.stop();

=> 'changing to the stopping state...'
=> 'we have entered the stopping state...'
=> 'stopping the video...'
```

Hopefully this gives you some food for thought the next time you're trying to wrangle some unwieldy component. I cannot count how many times I've used this pattern in other languages to tidy up and organize my code. I'm looking forward to working with it again in Backbone and I'd love to hear anyone's take on how this can be tweaked or improved. Till tomorrow! - Rob

You should follow me on Twitter [here.](http://twitter.com/rob_dodson)

- Time: 9:48 pm
- Mood: Focused, Hurried
- Sleep: 5
- Hunger: 0
- Coffee: 1
