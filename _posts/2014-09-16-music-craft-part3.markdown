---
layout: post
title:  "Hack Week project: music-craft part-3. Textures and movement"
date:   2014-09-16 12:09:23
categories: games
---

### adding some movement

The first thing I added was some basic movement. Ideally I would have liked this to be controlled largely by the mouse. However it's not immediately clear how to tie this in to the way that play-clj processes inputs. It would require locking the mouse cursor and making it invisible. Even using the java interop I couldn't tie this into the library. I've asked the question on the [play-clj subreddit](http://www.reddit.com/r/playclj/) and I'll see if I get a response. For the moment I'm using the keyboard as an alternative.

### key handler

Moving the camera with the keyboard is simple enough

{% highlight clojure %}
:on-key-down
(fn [{:keys [key] :as screen} entities]
  (condp = key
     (key-code :w)
     (translate-camera screen 0 0 -1)))
{% endhighlight %}
<br>

We simply add a `:on-key-down` handler and take the keys from the `screen` map. The provided `(key-code)` function gives us a key that matches those in the keys map.

Translate camera is a simple function that moves the camera on the x, y and z axis before calling update on the perspective to 'commit' the changes.

{% highlight clojure %}
(defn translate-camera
  "Translate the camera on the x, y and z axis"
  [screen x y z]
  (doto screen
    (perspective! :translate x y z)
    (perspective! :update)))
{% endhighlight %}

One small mistake I made here was to not return the entities from the :on-key-down handler. This resulted in my cube disappearing. My first thought was that I was moving the camera incorrectly and that I just couldn't see the cube any more. Eventually I realised that I was returning nil which was being interpreted as an empty list of entities. 

### textures

Next up was adding some texture to the cube. I ended up doing this with java interop as the support for adding textures to materials didn't seem to be present in play-clj (I may well be wrong about that).

{% highlight clojure %}
(defn raw-texture
  "Creates a libgdx texture from the supplied resourece path"
  [path]
  (-> path io/resource .toURI java.io.File. FileHandle. Texture.))

(def grass-texture (delay (raw-texture "grass.jpg")))

(defn random-texture
  "Returns a random texture"
  []
  (rand-nth [grass-texture stone-texture sand-texture water-texture]))

(defn block
  "Creates a block at pos x, y, z with a random texture"
  [x y z]
  (let [texture-attr (TextureAttribute. TextureAttribute/Diffuse @(random-texture))
        model-mat (material :set texture-attr)
        ...snip...
{% endhighlight %}
<br>

`raw-texture` loads the file as a resource before creating a libgdx `FileHandle` which is passed to the texture.

Each texture is created as a delay as a texture can only be created inside a thread that contains the opengl context. This will be true when called from the gameloop but not when the code is evaluated. A delay solves this problem for us but does mean that we need to deref it to get the value.

Only a few changes are required to generate the block, we just have to create a texture attribute rather than a color attribute and add this when we create the material.

### more blocks!

As you might be able to gather from the code above I addeda block function that creates a block when supplied some coordinates. I tied this into an extremely basic function that generates a 20 x 20 x 1 grid of blocks.

{% highlight clojure %}
(defn blocks
  "Creats a 1 deep plane of blocks from +20 to -20 on the x and z axis"
  []
  (vec (for [x (range (- grid-size) grid-size 2)
             z (range (- grid-size) grid-size 2)]
         (block x 0 z))))
{% endhighlight %}
<br>

### how does it look?

<a href="/img/music-craft/part3-block-grid.png"><img src="/img/music-craft/part3-block-grid.png" alt="block grid" width="500px" /></a>

Things are starting to look a touch more interesting now.

### which tag?

Clone the repo and run:

`git checkout tags/cube-grid`

To see the code as it is right now

### what's next?

World generation. I've decided to start looking into world generation a little first before tying the music into it. Hopefully that will be a little simpler than doing it in one pass.
