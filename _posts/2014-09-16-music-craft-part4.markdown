---
layout: post
title:  "Hack Week project: music-craft part-4. World generation"
date:   2014-09-16 12:09:23
categories: games
---

### just how do you generate a world anyway?

That's a good question and there are no doubt many different ways to go about it. The wikipedia page on [procedural generation](prochttp://en.wikipedia.org/wiki/Procedural_generation) is actually quite a good place to start looking, it gives you the right terminology to google anyway. Supposedly Minecraft now uses a system of fractal generation but [originally](http://notch.tumblr.com/post/3746989361/terrain-generation-part-1) it used perlin noise functions. Whilst the actual details are a bit thin the idea of using a noise function is enough to get going with.

### perlin noise functions

Perlin noise was actually a new concept to me. Fortunately I managed to find [an excellent write up](http://webstaff.itn.liu.se/~stegu/TNM022-2005/perlinnoiselinks/perlin-noise-math-faq.html) It's a really clever trick!

Rather than re-implementing it myself I was able to find a [Clojure library](https://github.com/mikera/clisk) that generates 2D noise when supplied with two values between 0 and 1. It also allows you to set the seed which could be useful if I ever need to start dealing with chunking.

So this gives us a smooth graduation of "random" values between 0 and 1 over a 2D axis.

We can take these x and y noise values and multiply them by a height to provide us with a smoothly graduating terrain of hills.

Here's my code with some explanatory comments:

{% highlight clojure %}
(defn noise-for-grid
  "Returns a grid of noise values between min and max for grid x-size and y-size"
  [x-size y-size min max seed]
  ;; set the seed
  (Simplex/seed seed)
  ;; create a 2D array to update the noise values in
  (let [output (make-array Integer/TYPE x-size y-size)]
    ;; go through each x and y in the grid size
    (doseq [x (range 0 x-size)
            y (range 0 y-size)]
      ;; normalise x and y to be between 0 and the maximum grid size
      (let [nor-x (/ x x-size)
            nor-y (/ y y-size)
            ;; generate noise for the position x, y
            noise (Simplex/noise nor-x nor-y)
            ;; scale the noise value by the maximum height
            r-height (Math/round (* noise (- max min)))
            ;; add the minimum height
            min-height (+ min r-height)]
        ;; update the output array with the new noise value
        (aset output x y (Integer. min-height))))
    output))
{% endhighlight %}
<br>

We can then tie this noise grid back into the block generation function:

{% highlight clojure %}
(defn blocks
  "Create the block entities"
  []
  (let [seed 1337
        min 1
        max 10
        noise (noise-for-grid grid-size grid-size min max seed)]
    (vec (for [x (range grid-size)
               z (range grid-size)
               y (range min (aget noise x z))]
           (block x y z)))))
{% endhighlight %}
<br>

There isn't too much change here, we just additionally iterate over the y axis within the bounds of the height specified by the noise function.

### how does it look?

<a href="/img/music-craft/part4-world.png"><img src="/img/music-craft/part4-world.png" alt="world image" width="500px" /></a>

Pretty cool! That wasn't too hard once I'd had the idea.

### what's the git tag?

Clone the repo and run:

`git checkout tags/noise-generation`

To see the code as it is right now

### what's next?

Performing some additional passes over the world, probably with another noise function to generate a surface texture.
