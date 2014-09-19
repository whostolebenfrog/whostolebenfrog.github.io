---
layout: post
title:  "Hack Week project: music-craft part-6. Adding some music"
date:   2014-09-18 10:09:23
categories: games
---

### running out of time

My plan was to do the audio analysis myself which is what I started off doing. Unfortunately time was coming to an end I was still a long way from having it working. As such I decided to use [Echonest's excellent audio analysis api](http://developer.echonest.com/docs/v4/track.html) to quickly grab some data about my 3 example songs. I'm a bit disappointed but I wanted to get something visual to show at the end of week demos. It's something I will need to continue working on.

### playing the songs

Playing the music is really simple. Just place the songs in the resources folder and use `(sound)`. I keep track the resulting audio object in an atom so that 

{% highlight clojure %}
(when @current-track
  (stop-track)
  (reset! playing-track (sound (:file @current-track) :play)))
{% endhighlight %}
<br>

We can then stop the playing with

{% highlight clojure %}
(when @current-track
(defn stop-track
  "stop playing the current track"
  []
  (when @playing-track
    (sound! @playing-track :stop)
    (reset! playing-track nil)))
{% endhighlight %}
<br>

### altering the world

I've saved the echonest data in a .info file inside of the resources folder. Loading json into a clojure map is really simple, typically at [MixRadio](http://dev.mixrad.io) we use the wonderful [cheshire](https://github.com/dakrone/cheshire) library to parse JSON. For this I tried out [pjson](https://github.com/gerritjvv/pjson) as I've been looking at it for performance. No big problems switching over although I miss the lack of automatic conversion of strings to keywords. Anyway reading the json is as simple as:

{% highlight clojure %}
(when @current-track (-> (:info @current-track) io/resource slurp json/parse-string))
{% endhighlight %}
<br>

Calling `:info` on the @current-track simply retrieves the filename. That gives me a nice Clojure map to work with.

We pass this map into the `(blocks)` function in the world namespace to provide variation for our generation.

{% highlight clojure %}
(defn blocks
  "Create the block entities"
  [info]
  (let [energy (info "energy")
        seed (info "danceability")
        min 1
        max (Math/round (* energy 15))
        noise (noise-for-grid grid-size grid-size min max seed)]
    (vec (for [x (range grid-size)
               z (range grid-size)
               y (range 0 (aget noise x z))]
           (block x y z energy)))))
{% endhighlight %}
<br>

The `(blocks)` function now takes the danceability attribute as seed for the simplex noise (just to provide variation) and makes use of the energy attribute (which is between 0 and 1) to scale the height of the world. We also pass the energy value into the `(block)` function to vary the block textures that are associated with the type of song.

{% highlight clojure %}
(def random
  "Our random number generator"
  (java.util.Random.))

(defn weighted-gaussian-random
  "Returns a weighted normally distributed random number between 0 and 1"
  [weight]
  (min (* weight (Math/abs (.nextGaussian random))) 1))

(defn nearest-material
  "Find the nearest material to the supplied number"
  [n mats]
  (apply min-key (fn [{:keys [energy]}] (Math/abs (- n energy))) mats))


(defn random-material
  "Returns a random texture"
  [energy]
  (let [energy-mats [{:mat grass-material :energy 0.2}
                     {:mat stone-material :energy 0.6}
                     {:mat sand-material  :energy 0}
                     {:mat water-material :energy 0.3}
                     {:mat stone2-material :energy 0.8}
                     {:mat fire-material :energy 1}]]

    (:mat (nearest-material (weighted-gaussian-random energy) energy-mats))))
{% endhighlight %}
<br>

`(block)` really just passes the energy onto `(random-material)` which uses it to assign the material. I want to cluster the block types towards a type of world, e.g. grass for a calm song and fire for a heavy song. To do this I'm generating the random number using a normal distribution weighted by the energy. This allows most blocks to be clustered around a point but with some deviation.

<a href="/img/music-craft/normal-distribution.png"><img src="/img/music-craft/normal-distribution.png" alt="world image" width="500px" /></a>

Next I use Clojure's `(min-key)` function to find the closest block (in terms of energy) to the random number I generated. This is shown above in the `(nearest-material)` function.

The calm world from an ambient song:

<a href="/img/music-craft/part6-calm.png"><img src="/img/music-craft/part6-calm.png" alt="world image" width="500px" /></a>

The dangerous world from a metal song:

<a href="/img/music-craft/part6-dangerous.png"><img src="/img/music-craft/part6-dangerous.png" alt="world image" width="500px" /></a>

### adding a quick menu

Finally I wanted to add a quick menu screen to load the songs.

{% highlight clojure %}
(defn rotate-menu
  [entities direction]
  (let [menu-items (count (filter :menu? entities))
        old-active @menu-selected
        new-active (reset! menu-selected (mod (direction @menu-selected) menu-items))]
    (for [entity entities]
      (if (:menu? entity)
        (condp = (:pos entity)
          old-active (-> (assoc-in entity [:active?] false)
                         (doto (label! :set-color (color :white))))
          new-active (-> (assoc-in entity [:active?] true)
                         (doto (label! :set-color (color :green))))
          entity)
        entity))))

(defn menu-option
  [pos text & [{:keys [menu-color menu? active? file info]
                :or {menu-color (color :white)
                     menu? true
                     active? false}}]]
  (assoc (label "0" menu-color :set-text text)
    :menu? menu? :pos pos :x 5 :y (* pos 20) :active? active? :file file :info info))

(defscreen menu-screen
  :on-show
  (fn [screen entities]
    (update! screen :camera (orthographic) :renderer (stage))
    (input! :set-cursor-catched true)
    [(menu-option 0 "ambient.mp3" {:file "ambient1.mp3" :info "ambient1.info"})
     (menu-option 1 "metal.mp3"   {:file "metal1.mp3"   :info "metal1.info"})
     (menu-option 2 "blues.mp3"   {:file "blues1.mp3"   :info "blues1.info" :active? true })
     (menu-option 3 "-----------" {:menu? false})
     (menu-option 4 "music-craft" {:menu-color (color :red)
                                   :menu? false})])
{% endhighlight %}
<br>

This is just another `(screen)` but with some code to create and highlight menu items, when they are rotated by a key event. The key event handler just calls rotate-menu with the list of entities and either `(inc)` or `(dec)`. I don't love using the `(atom)` here but it's difficult to pass state around in play-clj and doing without needlessly complicates the code. It's the pragmatic choice I suppose.

<a href="/img/music-craft/part6-menu.png"><img src="/img/music-craft/part6-menu.png" alt="world image" width="500px" /></a>

### where's the code?

Clone the repo and run:

`git checkout tags/music`

To see the code as it is right now

### what's next

Well I'm out of time for this hack project but there are a few things I'd like to continue with. Firstly I want to get my own audio analysis working to replace the echonest data. Secondly I'd like to rewrite the rendering so that it's significantly more efficient and continue with the world generation as I think it's an interesting example of using clojure. Finally I'd like to write up my thoughts on how well libgdx and play-clj worked in this project.
