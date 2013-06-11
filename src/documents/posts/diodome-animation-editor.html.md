---
title: Making the Diodome Animation Editor
layout: post
tags: ['led','arduino','node.js', 'diodome', 'mongodb']
date: 06/04/2013
ignored: true
---
for all this has been to figure out how we're going to control the 300+ LEDs that will make up the dome and how we can develop animations and interactive experiences for it without having a giant dome setup in my spare room!

## Lets go 3d...

The best way I could think of to make animations without having the whole dome and all the physical hardware setup was to create a 3d simulation of the dome. This wasn't as hard as I had feared since I'd already made up a model of the dome using Blender for the [fundraising campaign](http://igg.me/at/diodome). I ended up using [THREE.js](http://threejs.org) to render the dome model in the browser and then used [CodeMirror](http://codemirror.net/) to create a little code editor window which could be used to write animations to control it. You can [try it out here](http://diodome.heroku.com).

But how will I use all these nice animations on the physical hardware? I don't want to have to rewrite all the work just to deploy it on an Arduino. Thankfully this is where Node.js comes in and makes itself incredibly useful. What we're going to do is run Node.js on a Raspberry Pi, which will then control the LEDs!