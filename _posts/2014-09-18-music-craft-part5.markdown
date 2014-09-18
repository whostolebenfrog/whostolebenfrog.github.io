---
layout: post
title:  "Hack Week project: music-craft part-5. Fixing a few things"
date:   2014-09-18 10:09:23
categories: games
---

### performance

Firstly I needed to add in a few basic performance tweaks. The performance in this demo is always going to suck a little as I'm not making use of any clever tricks to avoid rendering blocks. In a game like this you actually only ever need to render a tiny subset of the blocks around you in the world. Blocks hidden by other blocks account for the vast majority of the world (e.g. underground) and even of the visible blocks only a few faces usually need to be rendered. However building a clever system takes a fair bit more time than a simple system and it's not really the purpose of this demo. As such I'm just going to add a few little tweaks.

#### engine tweaks

Firstly we can tell the engine not to render the back faces of blocks. That's as simple as setting a couple of open-gl params so we'll do that.

Secondly we can switch off continuous rendering. This tells the game not to bother re-rendering if the game state hasn't changed e.g. we're not moving. Whilst this doesn't actually increase the speed of the game when it's actually needed it does mean that whilst I'm just messing around coding, the game isn't constantly re-rendering in the background. That's pretty useful on my crappy home laptop, it stop it from burning a hole in my leg anyway.

{% highlight clojure %}
(let [gl2 (graphics! :get-g-l20)]
  (-> gl2 (.glEnable GL20/GL_CULL_FACE))
  (-> gl2 (.glCullFace GL20/GL_BACK)))
(graphics! :set-continuous-rendering false)
{% endhighlight %}
<br>

### view frustum culling

This is just about the simplest actual performance tweak we can do. Basically the idea is that if an object is behind the camera we don't render it. All that takes is a simple filter over the set of all entities and the use of a built-in function to tell us if it's behind the frustum.

{% highlight clojure %}
(let [frus (.frustum (u/get-obj screen :camera))
      visible-entities (filter (fn [{:keys [x y z]}] (.pointInFrustum frus x y z)) entities)]
  (reset! visible-objects (count visible-entities))
  (render! screen visible-entities))
{% endhighlight %}
<br>

We do this inside the :on-render handler, taking care not to forget to return the full set of entities. A similar technique could be used to decide whether or not to render whole chunks which would allow you to handle much bigger worlds.

### mouse movement

Contrary to what I said a few posts back, it is possible to use play-clj to capture the whole mouse movement. Fortunately the play-clj creator [oakes](http://www.reddit.com/user/oakes) was kind enough to help me out and answer my question. It turns out there is an (input!) function that I had missed which handles everything you need here. It's enabled in the :on-show handler using:

{% highlight clojure %}
(input! :set-cursor-catched true)
{% endhighlight %}
<br>

I also added a toggle when the user presses the 't' key.

Handling the mouse move requires that we deal with pitch and yaw according to the mouse movements. Moving the mouse left and right is pretty simple as we can just rotate the camera around the y axis. However up and down is more complicated as which axis we rotate around depends on which direction we are facing. This means that we need to rotate around the cross product of the cameras direction and the y vector. This gives us what's known as an FPS-like camera. This is actually a bigger area than I would have thought. Fortunately I found an [excellent writeup](http://www-rohan.sdsu.edu/~stewart/cs583/LearningXNA3_lects/Lect15_Ch11_CreateFirstPersonCamera.html) that explains the need for the cross-product and the various different types of camera that you might need to use.

{% highlight clojure %}
(defn handle-mouse-move
  [screen]
  (let [camera (u/get-obj screen :camera)
        delta-x (* (input! :get-delta-x) degress-per-pixel)
        delta-y (* (input! :get-delta-y) degress-per-pixel)
        vec-y (doto (Vector3.) (.set (.direction camera)) (.crs Vector3/Y))]
    (doto screen
      (perspective! :rotate Vector3/Y delta-x)
      (perspective! :rotate vec-y delta-y)
      (perspective! :update))))
{% endhighlight %}
<br>

### keyboard movement

All that's left is that pressing the forward key moves you in the direction of the camera.

{% highlight clojure %}
(key-code :w)
(do
  (perspective! screen :translate (.direction (u/get-obj screen :camera)))
  (perspective! screen :update))
{% endhighlight %}
<br>

Which is simple enough

### what's the git tag?

Clone the repo and run:

`git checkout tags/movement-performance`

To see the code as it is right now

### what's next

It's probably about time I introduced some music.
