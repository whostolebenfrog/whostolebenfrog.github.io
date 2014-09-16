---
layout: post
title:  "Hack Week project: music-craft part-2. Rendering"
date:   2014-09-15 10:09:23
categories: games
---

### rendering a cube

At the end of the last section I said I wanted to be able to render a single 3D cube. Well that now works. I'll briefly talk through the code in this section. I took most of this from the [minimal 3D example](https://github.com/oakes/play-clj-examples/) in the play-clj repo.

### how do I run it?

The project folder contains 3 directories, desktop, iOS and android. Navigate to the desktop directory and you will see a lein project.clj. From this directory start a repl with lein repl and it will sort out your dependencies and do some compilation.

There are two important files:

    src
    └── music_craft
        └── core
            └── desktop_launcher.clj

Which contains the -main method

and

    src-common/
    └── music_craft
        └── core.clj

Which contains your code.

From the desktop_launcher.clj namespace eval the main method

`(-main)`

This will open a window with a single, green, 3D block displayed.

<a href="/img/music-craft/part2-single-block.png"><img src="/img/music-craft/part2-single-block.png" alt="cube example" width="500px" /></a>

I've included a line at the bottom of core.clj

{% highlight clojure %}
(on-gl (set-screen! music-craft main-screen))
{% endhighlight %}
<br>

That reloads the whole screen and allows you to dynamically make changes to the code and have them reflected instantly in the running window. All you need to do is re-evaluate the namespace. Without that only changes in the :on-render function will be applied.

### Looking at the code

So, briefly, what does this code do? The examples don't provide any context so I'll try and document a few things here.

{% highlight clojure %}
    ;; define a new screen where we can render some stuff, this will be a 3D screen
    (defscreen main-screen
      ;; this is run on init, takes a list of entities and returns an updated list
      ;; of entities
      :on-show
      (fn [screen entities]
        (update! screen
                 ;; define the renderer to use batch approach
                 :renderer (model-batch)
                 ;; define a single ambient light for the scene
                 :attributes (let [attr-type (attribute-type :color :ambient-light)
                                   attr (attribute :color attr-type 0.4 0.4 0.4 1)]
                               (environment :set attr))
                 ;; set up the camera with a 67 degree field of view, aspect ratio to fit
                 ;; the window size (note this doesn't handle resizing yet)
                 ;; 
                 ;; the position is set to be 5 units in x, y and z facing towards 0, 0, 0
                 ;; near! sets the near clipping pane distance and far! the far clipping
                 ;; distance
                 :camera (doto (perspective 67 (game :width) (game :height))
                           (position! 5 5 5)
                           (direction! 0 0 0)
                           (near! 0.1)
                           (far! 300)))
        ;; here we create the box that we are rendering
        ;; the bit-or creates the bitmask that sets the two attribute properties
        ;; position and normal required to get shape rendered and get lighting working
        ;; finally we set the x, y and z coords
        (let [attr (attribute! :color :create-diffuse (color :green))
              model-mat (material :set attr)
              model-attrs (bit-or (usage :position) (usage :normal))
              builder (model-builder)]
          (-> (model-builder! builder :create-box 2 2 2 model-mat model-attrs)
              model
              (assoc :x 0 :y 0 :z 0))))
    
      ;; this gets called on each frame, we clear the screen and render the
      ;; vector of entities
      :on-render
      (fn [screen entities]
        (clear! 1 1 1 1)
        (render! screen entities)))
    
    ;; Define's the game and set's the screen from above 
    (defgame music-craft
      :on-create
      (fn [this]
        (set-screen! this main-screen)))
{% endhighlight %}


### which tag?

Clone the repo and run:

`git checkout tags/single-cube`

To see the code as it is right now

### what's next

Directional lighting to add some spark, movement and rendering some more cubes!
