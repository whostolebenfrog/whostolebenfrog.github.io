---
layout: post
title:  "Interview question: Finding the largest contiguous array"
date:   2014-06-27 12:24:23
categories: interviews
---

I was reading [a list of popular interview questions](http://www.ocf.berkeley.edu/~kelu/interviews/questions.html) and one of them jumped out to me. Actually, at the time of my writing this, I've only made it this far down the list as I had to stop and try to solve it.

The question is:

<code>Given an array of positive and negative integers. Find the starting index and length of sub-array with the largest sum. 

Can be done in linear time with constant memory.</code>

It's not a question I've seen before and the answer didn't immediately jump to mind. So how do we go about solving it? There is a good clue above in that it can be solved in linear time. We probably shouldn't try something totally naive like testing every sub-array.

When faced with a question like this it can be helpful to list what we know and, if in an interview, clarify any assumptions. Here's what I came up with:

* Sub-arrays have to be contiguous
* The winning array may include negative numbers as the next number along might more than make up for the loss. e.g. `[3, 2, -2, 7]`. In this case it's worth taking the `-2` to gather up the positive `7`.
* We need to keep track of the best sum that we've found so far, as well as some indexes and counts
* If the sum of the items in the current sub-array goes below 0 then there is no point continuing to expand it. Anything afterwards will always be bigger without it
* The array contains both positive and negative integers which means that it must have a length and that the final result must be greater than 0, this lets us skip some error cases

At this point I had an inkling of what the answer would look like:

1. Loop through the numbers keeping track of the `currentSum` and `maxSum`
2. If the `currentSum` is `> 0` then keep going
3. If the `currentSum` is bigger than `maxSum` then update `maxSum`
4. If the `currentSum` is less than `0` we set it back to `0` and reset
5. Keeping the index and length of the sub-array adds a little book keeping but doesn't change the algorithm

That's not exactly exhaustive but it's enough to start writing some code. The specifics will likely sort themselves out during the implementation.

I wrote it in javascript as I had a browser open:

{% highlight javascript %}
var nums = [-4, 3, 0, -1, -6, 4, 5, -2, 3, -1, 2, -5, 4],
    currentMax = 0,
    currentCount = 0,
    currentStartIndex = 0,
    bestMax = 0,
    bestCount = 0,
    bestStartIndex = 0;

for (var i = 0; i < nums.length; i++) {
  var n = nums[i];
  currentMax += n;
  if (currentMax > 0) {
    if (currentCount === 0) {
      currentStartIndex = i;
    }
    ++currentCount;
    if (currentMax > bestMax) {
      bestMax = currentMax;
      bestStartIndex = currentStartIndex;
      bestCount = currentCount;
    }
  } else {
    currentMax = 0;
    currentCount = 0;
  }
}

// bestMax === 11
// bestStartIndex == 5
// bestCount == 6

{% endhighlight %}

<br>
Well, I don't think Crockford is going to be too happy but we're solving interview questions here and that appears to do the trick.

The key here is that we can treat going below `0` as a cue to stop reading the current sub-array. Besides this we are simply keeping a track of the highest sum that we've seen so far. Going below `0` just tells us when to abandon the current array. Keeping track of the start index and length makes the solution slightly more verbose but doesn't affect how we solve it.

TODO: how does this look in Clojure?
