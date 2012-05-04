---
layout: post
title: "D3 Basics: The Linear Scale"
date: 2012-05-04 07:21
comments: true
categories: [Chain, D3, Data Viz]
---

<!-- CSS Styles: -->
<div>
  <style type="text/css">

    .chart {
      font-family: Arial, sans-serif;
      font-size: 10px;
    }

    .bar {
      fill: steelblue;
    }

    .axis path, .axis line {
      fill: none;
      stroke: #000;
      shape-rendering: crispEdges;
    }

    .label {
      font-size: 12 px;
      fill: #FFF;
    }

    .point {
      stroke: #666;
      fill: red;
    }

  </style>
</div>
In [the last post](http://localhost:4000/blog/2012/05/03/d3-basics-an-introduction-to-scales/) we did a basic introduction to the concept of scales in [D3.js](http://d3js.org/). Today we'll look at our first scale and write some code to visualize it.

### Linear Scales

The most basic scale in D3 is the `linear scale` which maps a continous `domain` to an output range. To define a linear domain we'll need to first come up with a data set. Fibonacci numbers work well, so let's declare a variable `data` like so:

```js
var data = [1, 1, 2, 3, 5, 8];
```

The data set will represent our scale's input domain. The next step is defining an output range. Since we're going to be graphing these numbers we want our range to represent screen coordinates. Let's go ahead and declare a `width` and a `height` variable and set them to 320 by 150.

```js
var width = 320,
    height = 150;
```

We now have everything we need to create our first scale. 
```js
var x = d3.scale.linear()
        .domain([0, d3.max(data)])
        .range([0, width]);
```

D3 methods often return a value of `self` meaning you can chain method calls onto one another. If you're used to jQuery this should be a common idiom for you. You'll notice that both the domain and the range functions accept arrays as parameters. Typically the domain only receives two values, the minimum and maximum of our data set. Likewise the range will be given the minimum and maximum of our output space. You could pass in multiple values to create a polylinear scale but that's outside the scope of our dicussion today.

In the domain function we're using a helper called `d3.max`. Similar to `Math.max`, it looks at our data set and figures out what is the largest value. While `Math.max` only works on two numbers, `d3.max` will iterate over an entire `Array` for us. 

If you've been following along in your own file you should be able to open your console and type `x(8)` to get 300.

With just this information alone we have enough to build our first graph.

<!-- D3.js Chart -->
<small>Fibonacci Sequence Chart 1.0</small>
<div id="linear-scale-chart-1"></div>
<script type='text/javascript'>
(function() {

  var data = [1, 1, 2, 3, 5, 8];
  var width = 320
      height = 150;

  var x = d3.scale.linear()
          .domain([0, d3.max(data)])
          .range([0, width]);

  var svg = d3.select('#linear-scale-chart-1').append('svg')
          .attr('width', width)
          .attr('height', height)
          .attr('class', 'chart');

  svg.selectAll('.chart')
          .data(data)
        .enter().append('rect')
          .attr('class', 'bar')
          .attr('y', function(d, i) { return i * 20 })
          .attr('width', function(d) { return x(d); })
          .attr('height', 15);

})();
</script>

```js JavaScript
var data = [1, 1, 2, 3, 5, 8];
var width = 320
    height = 150;

var x = d3.scale.linear()
        .domain([0, d3.max(data)])  
        .range([0, width]);

var svg = d3.select('body').append('svg')
        .attr('width', width)
        .attr('height', height)
        .attr('class', 'chart');

svg.selectAll('.chart')
        .data(data)
      .enter().append('rect')
        .attr('class', 'bar')
        .attr('y', function(d, i) { return i * 20 })
        .attr('width', function(d) { return x(d); })
        .attr('height', 15);
```

```css CSS
.chart {
  font-family: Arial, sans-serif;
  font-size: 10px;
}

.bar {
  fill: steelblue;
}
```
### Break It Down
Let's go through the JavaScript piece by piece to outline what happened.

```js
var data = [1, 1, 2, 3, 5, 8];
var width = 320
    height = 150;

var x = d3.scale.linear()
        .domain([0, d3.max(data)])
        .range([0, width]);
```

The first block should be pretty familar at this point. We've declared our Fibonacci data, our explicit width and height and defined our scale. Nothing new here.

```js
var svg = d3.select('body').append('svg')
        .attr('width', width)
        .attr('height', height)
        .attr('class', 'chart');
```

In the next section we're declaring our `SVG` element. We use a D3 selection to grab the `body` tag and we append an `svg` tag onto it. Since D3 uses method-chaining we can then start assigning attributes to our SVG element. We declare the width and the height to match the explicit values we defined earlier and finally we give it a class name of `chart`.

```js
svg.selectAll('.chart')
        .data(data)
      .enter().append('rect')
        .attr('class', 'bar')
        .attr('y', function(d, i) { return i * 20 })
        .attr('width', function(d) { return x(d); })
        .attr('height', 15);
```

This last section is where it all ties together. Since we stored our SVG element in a variable called `svg` we're able to easily reference it again. We then instruct D3 to create a `join` by calling the `data` method and passing in our `Array` of values. When D3 performs a join it steps through each element in the array and attempts to match it to a figure that already exists on the page. If nothing exists it will call the `enter` function. At this point it steps through the array again, passing the values to us so we can define new shapes. [For a much more in-depth explanation of joins refer back to this article.](http://bost.ocks.org/mike/join/)

In our case we're appending SVG `Rects` but it could just as easily be circles or other shapes. We give each rect a class of `bar` so we can style it with CSS. When we declare the `y` attribute instead of using an explicit value we create an `accessor`, a little helper function which takes a piece of data and an optional index as its arguments. In this case `d` will equal subsequent elements in our data array and `i` will equal their indices. You can change that line to read like this:
```js
.attr('y', function(d, i) { console.log(d, i); return i * 20 })
```
for a much clearer picture of what's going on. Since we're just trying to space out our bars along the y-axis we don't really care about the value of `d`. Instead we want to know about the order of the bars, so we can say that data[0] should have a y of 0, data[1] should have a y of 20 (remember it's i * 20), data[2] should have a y of 40, etc.