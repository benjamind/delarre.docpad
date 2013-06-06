---
title: From Node to Diode
layout: post
tags: ['led','arduino','raspberry pi','node.js', 'diodome']
date: 06/04/2013
---
I have a confession to make. I'm an LED addict. If its got blinkenlights I'm your man. In 2010 I helped build the [Illuminatrix](http://cwd.co.uk/illuminatrix), a 4ft x 4ft wall of LEDs nestled inside ping pong balls that displayed hundreds of animations from people all around the world. This was my first big electronics project, besides a few kits I'd built and some awful attempts at implementing various audio syntheziser circuits I had found online. But I officially caught the hacker bug back then and that project inspired a lot of my work over the last few years.

This year we decided to build another project for the Burning Man festival. We're calling it the [Diodome](http://diodome.heroku.com) and its a 18ft geodesic dome containing hundreds of LEDs. Obviously.

I'll be detailing the build on my blog as we go, but the first step for all this has been to figure out how we're going to control the 300+ LEDs that will make up the dome and how we can develop animations and interactive experiences for it without having a giant dome setup in my spare room!

## Lets go 3d...

The best way I could think of to make animations without having the whole dome and all the physical hardware setup was to create a 3d simulation of the dome. This wasn't as hard as I had feared since I'd already made up a model of the dome using Blender for the [fundraising campaign](http://igg.me/at/diodome). I ended up using [THREE.js](http://threejs.org) to render the dome model in the browser and then used [CodeMirror](http://codemirror.net/) to create a little code editor window which could be used to write animations to control it. You can [try it out here](http://diodome.heroku.com).

But how will I use all these nice animations on the physical hardware? I don't want to have to rewrite all the work just to deploy it on an Arduino. Thankfully this is where Node.js comes in and makes itself incredibly useful. What we're going to do is run Node.js on a Raspberry Pi, which will then control the LEDs!

## Installing Node.js on Raspberry Pi

To get started we first had to compile Node for the pi, this turned out to be pretty simple.

```bash

```

This will take a long long time...for me almost 2 full hours. It should however go without a hitch.

Now that Node is running on the pi we'll need to figure out how to control the LEDs from node...

# Timing is essential

So the LEDs we're using for the Diodome project are the popular and inexpensive WS2811 based strings. These allow you to individually control hundreds of LEDs from a single GPIO pin, but they do require very precise timing in the region of 0.5 microseconds. This is an accuracy of timing that I'm just not sure could ever be accomplished from JavaScript, and likewise even the Raspberry Pi itself might struggle to bitbang pins with that degree of accuracy. Thankfully there's a simple way around this which makes all our lives a lot easier - use an Arduino.

Obviously an Arduino isn't powerful enough to run our animation code in Node, so we'll use the Raspberry Pi and then talk to the Arduino over the USB port. Thankfully the Arduino can talk to a Raspberry Pi using Serial over USB. All we need is to be able to write to the serial port from Node. Thankfully this is quite simple using the `node-serialport` library.

```javascript
codesample for node-serialport hereK
```

We can then write a simple sketch on the Arduino to read from the serial port, and set the color of the LEDs using the excellent NeoPixel library from Adafruit. This library has some rather excellent assembly code that can manage the accurate timing needed for the WS2811 drivers, and even manage to drive the LEDs at 800khz which gives us just enough time to update all our LEDs without any flicker. Here's the sketch I'm using on my Arduino Micro:

```c
codesample for arduino here
```

