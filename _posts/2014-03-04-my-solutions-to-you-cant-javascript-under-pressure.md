---
layout: post
title: My Solutions to "You Can’t JavaScript Under Pressure"
categories:
    - Development
redirect_from:
    - /my-solutions-to-you-cant-javascript-under-pressure/
---

A few months ago, there was a fun little quiz going around the web called "[You Can't JavaScript Under Pressure](http://games.usvsth3m.com/javascript-under-pressure/)" (if you want to take the quiz, go do it now before you read what I did). After taking the quiz, I saw some others had published their best responses and it seemed like a pretty good idea to share my responses as well. I read a few of these posts back then and knew my responses would be tainted by them, so I've waited a few months to forget what I had read and publish this.

**NOTE:** These responses are the best way I could come up with to solve the problems at hand, not necessarily the way I originally solved them when trying to do this time attack style. I've tried to note when I originally did it a different way, but since I first did it a few months ago, it's a little fuzzy.

### Problem #1
**Goal:** Build a function that takes an integer and double it.
<br />
**Test cases:** `2, 4, -10, 0, 100`

{% highlight javascript %}
function doubleInteger(integer) {
  return integer * 2;
}
{% endhighlight %}

Just some basic multiplication. Pretty simple.

### Problem #2
**Goal:** Build a function that determines if an integer is even or odd. It should return true if it's even and false if it's odd.
<br />
**Test cases:** `2, 3, 0, -2, Math.floor(Math.random()*1000000)*2)`

{% highlight javascript %}
function isNumberEven(integer) {
  return !(integer % 2);
}
{% endhighlight %}

Easiest solution is to use modulo. Using the ! (more often used this way as !!) operator is a little bit of a cheat here to get this function to return true or false instead of the remainder&mdash;it's a little bit of an anti-pattern as it can sometimes be tough to figure out what is going on, but this is a really simple function so I went ahead with it. I think the first time I did this I used a ternary operator instead of !.

### Problem #3
**Goal:** Build a function that when given a filename, returns the file extension if there is one. If there isn't a file extension, return false.
<br />
**Test cases:** `blatherskite.png, perfectlylegal.torrent, spaces are fine in file names.txt, this does not have one, .htaccess`

{% highlight javascript %}
function getFileExtension(filename) {
  var fileExtensionIndex = filename.lastIndexOf('.');

  if (fileExtensionIndex > -1) {
    return filename.substr(fileExtensionIndex + 1);
  }

  return false;
}
{% endhighlight %}

First, I found the location of the period that separated the file name and extension. Then, if there was a delimiter, I return all the text that exists after it. The first time I did this, I used a regular expression with a capture group. I can be a little quick to use regular expressions sometimes and although my first solution was a one-liner (if I remember correctly), I think this solution without regular expressions is much simpler.

### Problem #4
**Goal:** Build a function that when given an array finds the the longest string in the first level of the array.
<br />
**Test cases:** `[‘a’, ‘ab’, ‘abc’], [‘big’, [0, 1, 2, 3, 4], ‘tiny’], [‘Hi’, ‘World’, ‘你好’], [true, false, ‘lol’], [{ object: true, mainly: ‘to confuse you’ }, ‘x’]`

{% highlight javascript %}
function longestString(array) {
  var longString = '';

  array.forEach(function(word) {
    if ((typeof word === 'string') && (word.length > longString.length)) {
      longString = word;
    }
  });

  return longString;
}
{% endhighlight %}

This one just needs to iterate over each item in the array to see if it is longer than any string that we've looked at already. This remains a little simpler because we only need to check the first level of the array (which is different than example five), but we do need to do an explicit check to see if the item is a string since arrays also have a length property. We want to know the longest string, not which array has the most items!

### Problem #5
**Goal:** Build a function that will sum up all integers that exist in any level of a nested array.
<br />
**Test cases:** `[1, 2, 3, 4, 5], [[1, 2, 3], 4, 5], [[1, 2, false], '4', '5'], [[[[[[[[[1]]]]]]]], 1], [['A', 'B', 'C', 'easy as', 1, 2, 3]]`

This one was without a doubt the hardest one (and was the only one that required much thinking). I came up with two paradigms for how to do this and both require some recursion (at least in JavaScript), which can be a difficult thing to grasp at times.

#### First Solution to #5

{% highlight javascript %}
function flattenArray(array) {
  var flattenedArray = [];

  array.forEach(function(item) {
    if (Object.prototype.toString.call(item) === '[object Array]') {
      flattenedArray = flattenedArray.concat(flattenArray(item));
    }
    else {
      flattenedArray.push(item);
    }
  });

  return flattenedArray;
}

function arraySum(array) {
  return flattenArray(array).reduce(function(a, b) {
    return (typeof b === 'number') ? a + b : a;
  }, 0);
}
{% endhighlight %}

The solution I prefer:

1. flattens the array and
2. uses a reduce operation to add all the integers together

I've been spending some time in Ruby the last few months, and the way I would do this in Ruby makes this very easy to understand the JS version in my head:

{% highlight ruby %}
def arraySum(array)
  array.flatten.reduce(0) { |a, b| (b.is_a?(Integer)) ? a + b : a }
end
{% endhighlight %}

Unfortunately, JavaScript has no native flatten method for arrays and I wasn't using Lo-Dash or Underscore, so I had to build my own. The flatten function creates a new array and then iterates over the existing array. If the item it's iterating over is not an array, it will simply push it onto the new array. If it is an array, it will recursively call the flatten function again (and possibly again and again until it gets rid of all the layers) and then use the Array#concat method to merge the items that have already been added to the new array with the ones that returned from the recursive flatten call.

Once we have flattened the array that was input, we use a reduce operation to add everything together. JavaScript's Array#reduce (just referred to as reduce from here on out) isn't something you see very often in JavaScript, so I'll try to explain it a little bit in case anyone wasn't clear with how it works. Reduce works from left to right in an array (i.e. if given [1, 2, 3], it starts with 1 and then 2) and as arguments to the callback gives the result of the previous reduce operation (or the first item in the array if just starting out) and the next item in the array that needs to be reduced.

Seeing a really simple example often makes things easier to understand than reading about so here's a real simple example:

{% highlight javascript %}
// This would result in an answer of 6
// Works like ( (1 + 2) + 3 )
[1, 2, 3].reduce(function(a, b) {
  return a + b;
});
{% endhighlight %}

Getting back to the solution to problem #5, in the callback to reduce, I check to make sure the item we are attempting to add to previous result is a number&mdash;this avoids any type coercion issues that we might get with adding strings and numbers together. In addition, I supply an extra paramater of 0 to the reduce method. This sets the initial value to be 0, whereas without it, it would start with the first item in the array. Without this, I would have had to add additional checks to make sure that a first value of 'A' wouldn't ruin the whole process by giving me a result of 'A123' instead of 6.

#### Second Solution to #5

{% highlight javascript %}
function arraySum(array) {
  var sum = 0;

  array.forEach(function(item) {
    if (typeof item === 'number') {
      sum += item;
    }
    else if (Object.prototype.toString.call(item) === '[object Array]') {
      sum += arraySum(item);
    }
  });

  return sum;
}
{% endhighlight %}

The second solution to this problem still requires recursion but is a little bit easier to follow. We just iterate over each item in the array and try to add the numbers up. If we get an array instead of a number, we recursively call the same function until we just get a number out of that array (or nested array).

So this is how I solved these problems. If you hadn't taken the quiz before, how'd you do? Anyone think they have a better solution? Think I didn't explain something I did well enough? Let me know in the comments.
