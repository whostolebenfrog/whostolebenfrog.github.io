---
layout: post
title:  "Hack Week project: generating a 3D block world from music"
date:   2014-09-15 10:09:23
categories: games
---

### what's this?

As a very brief introduction this is Hack Week at [MixRadio](http://dev.mixrad.io). We have a whole week to work on whatever interests us. Last year I did something very practical and related to my work. This year I'm going to do the opposite.

My idea is to create a Minecraft like block world using various attributes of music as a seed. For example a relaxing song creates a flat and safe world. A discordant song creates a dangerous, mountainous map.

I'm intending to write this in Clojure, which is our language of choice at MixRadio, as I'm interested in seeing what a game looks like in such a non-traditional language.

### what's first?

First off I need to take a look at the various libraries available to me. A quick search reveals.

1. [lwjgl](http://lwjgl.org/)
2. [libgdx](http://libgdx.badlogicgames.com/)
3. [jmonkeyengine](http://jmonkeyengine.org/)

I'm not too worried about whether or not they have Clojure wrappers as Java interop in Clojure is excellent and it might be fun to start mocking out a bit of a library.

After a quick poke around libgdx looks like it should cover most of what I want to do. JMonkeyEngine seems like it's a bit higher level than I want. Lwjgl also looks great but I had to pick one and I don't have all the info to make an informed decision at this stage. LibGDX it is then. It also looks like LibGDX has a rather tasty clojure library called [play-clj](https://github.com/oakes/play-clj). It looks really well designed and should hopefully help provide some shape to my code.

### the code

`lein new play-clj music-craft`

That creates us a template and we push to github.

[music-craft repo on github](https://github.com/whostolebenfrog/music-craft)

I'll create tags at various points through development. Here is the first tag for the initial project creation.

`initial-upload`

### what's next?

I want to get a single untextured 3D box rendered into the world. Check out the next part for (hopefully) that.
