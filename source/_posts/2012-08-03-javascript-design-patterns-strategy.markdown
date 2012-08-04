---
layout: post
title: "JavaScript Design Patterns: Strategy"
date: 2012-08-03 15:12
comments: true
categories: [Chain, Design Patterns, JavaScript, Strategy]
---

The Strategy pattern is one of my personal favorites and you've probably seen or used it in some fashion without even knowing it. Its primary purpose is to help you separate the parts of an object which are subject to change from the rest of the static bits. Using Strategy objects versus subclasses can often result in much more flexible code since you're creating a suite of easily swappable algorithms.

## Formal Definition

{% blockquote GoF, Design Patterns: Elements of Reusable Object-Oriented Software %}
Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.
{% endblockquote %}

### Also Known As

- Policy

## Contrived Example Time!
Let's say you're making a game and you have a Character class. This game has all sorts of different terrain types so your character can run through open fields, walk slowly through swamps or swim under the ocean. Since you don't know what kind of other terrains the game designer is going to think up you decide that it would be a bad idea to give each character `run`, `walk`, and `swim` methods. After all, what if suddenly the character needs to `fly` or `crawl`? What if they're wounded and they need to `limp`? This situation could potentially get out of hand very fast...

There's a good chance you've seen or written code like this before:

``` js
function move() {
	if (state === 'walking') {
		// do some walk animation
	} else if (state === 'running') {
		// do some running animation
	} else if (state === 'swimming') {
		// do some swimming animation
	}
}
```

When you see a big conditional like that or a switch statement it's time to stop and wonder if there's a better way. For instance if we need to subclass our Character we're going to have to override that big conditional. What if we only want to replace the `swimming` bit? We'll have to copy and paste the code from the super class for `walking` and `running` and then write new code specifically for `swimming`. And of course if `walking` and `running` ever change we're totally screwed.

### We need a Strategy to deal with this

Ok so we know that our character is going to be a real contortionist and need to run and jump and crab-walk all over the place. What if we took the code that made her run and we put it in its own object? How about we just define a Class for movements and we do this for all the different kinds of motion? Then when we need to move our Character we'll just tell it to defer to one of these Movement objects.

``` js
var Movement = function(func) {
	this.move = func;
};

Movement.prototype.execute = function() {
	this.move();
};

var running = new Movement(function() {
	console.log("Hey I'm running!");
});

var walking = new Movement(function() {
	console.log("Just walking along...");
});
```

Now when we want to tell our character to move in a different way we'll just update which Movement object its currently referencing.

``` js
function changeMovementType(movement) {
	this.movement = movement;
}

function move() {
	this.movement.execute();
}
```

In practice you might have something that looks like this:

``` js
var running = new Movement(function() {
	console.log("Hey I'm running!");
});

var walking = new Movement(function() {
	console.log("Just walking along...");
});

// Create a hero and walk through a peaceful forest...

var hero = new Character();
hero.changeMovementType(walking);
hero.move();

// ... OH NO MOTHERFUCKIN' DINOSAURS!!!

hero.changeMovementType(running);
hero.move();
```

Now it's easy for us to add as many different kinds of motion as our little game designer can dream up. Want to give the character gas-powered robotic legs? No problem!

``` js
var robotlegs = new Movement(function() {
	console.log("Cruisin for oil...Look out humans!");
});

hero.changeMovementType(robotlegs);
hero.move();
```

## When to use it

When you have a part of your Class that's subject to change frequently or perhaps you have many related subclasses which only differ in behavior it's often a good time to consider using a Strategy pattern.

Another benefit of the Strategy pattern is that it can hide complex logic or data that the client doesn't need to know about.

## Real World Examples

Let's say you're designing a website with all kinds of different, interesting button animations. Sometimes buttons should expand when rolled over, other times they should fade and maybe some will change colors. You could have several button classes or jQuery selections but you'd be repeating a ton of code just for the sake of a little animation algorithm. Let's see if we can write some strategy objects which solve this problem for us and give us a nice toolbox to work from.

``` js
var ButtonAnimation = function(context, animation) {
	this.context = context;
	this.animation = func;
}
```

### Bocoup example

## Related Patterns

Flyweight: Strategy objects often make good flyweights.