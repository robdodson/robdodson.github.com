---
layout: post
title: "D3 Basics: Scales Pt. 1"
date: 2012-05-03 10:04
comments: true
categories: [Chain, D3, Data Viz]
---

### What Are Scales in D3?

A scale in D3 is a convenience function for mapping input data to an output range, typically x/y positions or width/height. Scales can also be used to link data to arbitrary values like categories or days of the week, or to quantize data into "buckets". 

There are two essential properties for any scale: the `domain` and the `range`.

#### Domain
The `domain` defines the input that the scale accepts. The `domain` can be *continuous* meaning it accepts essentially limitless data (ex: any number, any `Date`, any timestamp, etc.) or it can be *discrete* meaning it only accepts a certain set of data (ex: the numbers 1-10, every day in March).

#### Range
A `range` defines the potential output that the scale can generate. Values from the domain are mapped to values in the range. Let's look at two examples to help clarify.

Say you have an *identity scale*, meaning whatever value you use for input will also be returned as the output. We'll call the scale `x`. If `x` has a `domain` of 0 - 100 and a `range` of 0 - 100, then your `scale of 1` will be 1. Your `scale of 50` will be 50. Here's another way to write it:

```javascript
x(50) // returns 50
```

Now let's change the scale a bit. Let's say that `x` still has a `domain` of 0 - 100 but now it has an output `range` of only`0 - 10`. What do you think our `scale of 50` will return?
```javascript
x(50) // returns 5
```
Since we limited our potential output down to any number between 0 and 10 it narrowed the mapping from our `domain` to our `range`.

!!!! console example !!!!

In D3 a scale is both a `Function` and an `Object`. You can either call a scale—`x(100)`—or set a property to change its behavior: x.range([0, 100]).

D3 categorizes scales as either quantitative or ordinal. A quantitative scale is meant for any data set that has continuous input

### Quantitative Scales

  #### Linear Scales

  #### Time Scales

### Ordinal Scales

x = d3.scale.ordinal()
    .domain([10, 20, 30, 40])
    .range(['Group A', 'Group B', 'Group C', 'Group D']);

x = d3.scale.ordinal()
    .domain([new Date(2012, 0, 1), new Date(2012, 0, 5)])
    .range(['Group A', 'Group B', 'Group C', 'Group D']);