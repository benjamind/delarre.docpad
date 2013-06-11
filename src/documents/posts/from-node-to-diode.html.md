---
title: From Node to Diode
layout: post
tags: ['led','arduino','raspberry pi','node.js', 'diodome']
date: 06/04/2013
---
I have a confession to make. I'm an LED addict. If it involves making some blinkenlights I'm your man. In 2010 I helped build the [Illuminatrix](http://cwd.co.uk/illuminatrix), a 4ft x 4ft wall of LEDs nestled inside ping pong balls that displayed hundreds of animations from people all around the world. This was my first big electronics project, besides a few kits I'd built and some awful attempts at implementing various audio syntheziser circuits I had found online. But I officially caught the hacker bug back then and that project inspired a lot of my work over the last few years.

This year we decided to build another project for the Burning Man festival. We're calling it the [Diodome](http://diodome.heroku.com) and its an 18ft geodesic dome containing hundreds of LEDs. Obviously.

I'll be detailing the build on my blog as we go, but the first step is to figure out how we're going to control all these LEDs!

## How we did this before...

One of the things I did when I built the Illuminatrix was to build a web based animation editor that anyone could use to easily create animations for the project. In that you could write JavaScript code that we could use to generate keyframe data for the animations. We took the keyframe data and dumped it on an SD card and had a PIC microcontroller and custom circuit boards read from the SD card then control the LEDs directly. This worked well enough, but was a lot of work, and wasn't very stable or flexible. We could only use keyframed data, and we couldn't modify the animations on the fly.

So since we want to make the Diodome a lot more interactive and long term extendable than that we're going to go for another approach. I still want a web based animation editor, purely because I can't have an 18ft dome setup in my living room! So how can we take the JavaScript code from the editor and use it to control the LEDs without having to rewrite every animations code by hand to run on the hardware?

Enter NodeJS and the Raspberry Pi. The Pi is a wonderfully capable little machine, and thankfully Node compiles very well on it. So could we take the JavaScript based animations from the animation editor and run them on the Raspberry Pi and then use the Pi to control the LED strings?

## Installing Node.js on Raspberry Pi

To get started we first had to compile Node for the pi, this turned out to be pretty simple.

```bash
wget http://nodejs.org/dist/v0.10.9/node-v0.10.9.tar.gz
tar xvzf node-v0.10.9.tar.gz
cd node-v0.10.9
./configure && make && sudo make install
```

This will take a long long time...for me almost 2 full hours. It should however go without a hitch. You can check its working by doing:

```bash
node --version
npm --version
```

Now that Node is running on the pi we'll need to figure out how to control the LEDs from node...

# Timing is everything

So the LEDs we're using for the Diodome project are the popular and inexpensive WS2811 based strings you can get cheaply on Ebay or in our case direct from China. These allow you to individually control hundreds of LEDs from a single GPIO pin, but they do require very precise timing in the region of 0.5 microseconds. This is an accuracy of timing that I'm just not sure could ever be accomplished from JavaScript, and likewise even the Raspberry Pi itself might struggle to bitbang pins with that degree of accuracy. Thankfully there's a simple way around this which makes all our lives a lot easier - use an Arduino.

Obviously an Arduino isn't powerful enough to run our animation code in Node, so we'll use the Raspberry Pi and then talk to the Arduino over the USB port. Thankfully the Arduino can talk to a Raspberry Pi using Serial over USB. All we need is to be able to write to the serial port from Node. Thankfully this is quite simple using the `node-serialport` library. Here's a simple bit of code that opens up a serial port to the arduino and sends an array of color values for 5 LEDs.

```javascript
var serialport = require("serialport"),
	SerialPort = serialport.SerialPort;

// first list the serial ports available so we can figure out which is the arduino
serialport.list(function (err, ports) {
	var port = null;
	ports.forEach(function(p) {
		// this should work on windows and maybe osx
		if (p.manufacturer.indexOf('Arduino')!==-1) {
			port = p.comName;
		} else {
			// this will work on raspberry pi / linux
			if (p.hasOwnProperty('pnpId')){
				// FTDI captures the duemilanove //
				// Arduino captures the leonardo //
				if (p.pnpId.search('FTDI') != -1 || p.pnpId.search('Arduino') != -1) {
					port = p.comName;
				}
			}
		}
	});

	// port should now contain a string for the com port
	// open the port
	var serialPort = new SerialPort(port, {
		baudrate: 115200
	});
	// hook up open event
	serialPort.on("open", function () {
		// port is open
		console.log('port ' + port + ' opened');
		// hook up data listener to echo out data as its received
		serialPort.on('data', function(data) {
			console.log('data received: ' + data);
		});

		// here's an array of LED color values, 3 bytes per LED.
		var LEDS = [
			255,0,0,
			0,0,0,
			0,0,0,
			0,0,0,
			0,0,255
		];

		// create a Buffer object to hold the data
		var buffer = new Buffer(LEDS);
		// write it on the port
		serialPort.write(buffer, function(err, results) {
			if (err) {
				console.log('err ' + err);
			}
			console.log('wrote bytes : ' + results);
		});
	});
});
```

We can then write a simple sketch on the Arduino to read from the serial port, and set the color of the LEDs using the excellent [NeoPixel library from Adafruit](https://github.com/adafruit/Adafruit_NeoPixel). This library has some rather excellent assembly code that can manage the accurate timing needed for the WS2811 drivers, and even manage to drive the LEDs at 800khz which gives us just enough time to update all our LEDs without any flicker. Here's the sketch I'm using on my Arduino Micro:

```cpp
// include the neo pixel library
#include <Adafruit_NeoPixel.h>

// how many leds in our string?
static const int NUM_LEDS = 5;

// Parameter 1 = number of pixels in strip
// Parameter 2 = pin number (most are valid)
// Parameter 3 = pixel type flags, add together as needed:
//   NEO_RGB     Pixels are wired for RGB bitstream
//   NEO_GRB     Pixels are wired for GRB bitstream
//   NEO_KHZ400  400 KHz bitstream (e.g. FLORA pixels)
//   NEO_KHZ800  800 KHz bitstream (e.g. High Density LED strip)
Adafruit_NeoPixel strip = Adafruit_NeoPixel(NUM_LEDS, 6, NEO_GRB + NEO_KHZ400);

// buffer to hold colors of our LEDs
char colorValues[NUM_LEDS*3];

void setup() {
  strip.begin();
  strip.show();

  // initialize to black (off)
  for (int i=0; i < NUM_LEDS*3; i++) {
    colorValues[i] = 0;
  }
  
  // initialize the strip to the current values
  for(int i=0; i<NUM_LEDS; i++) {
    int d = i*3;
    uint32_t c = strip.Color(colorValues[d], colorValues[d+1], colorValues[d+2]);
    strip.setPixelColor(i, c);
  }
  // update the strip
  strip.show();

   //Initialize serial and wait for port to open:
  Serial.begin(115200); 
  while (!Serial) {
    ; // wait for port
  }
}

void loop() {
  // wait for bytes on serial port
  if (Serial.available() > 0) {
    // read 3 bytes per LED from serial port
    char bytesRead = Serial.readBytes(colorValues, NUM_LEDS*3);
    // check we got a full complement of bytes
    if (bytesRead < NUM_LEDS*3) {
      // something went wrong, abandon this loop
      return;
    }
    // feed the data to the leds
    for(int i=0; i<NUM_LEDS; i++) {
      int d = i*3;
      uint32_t c = strip.Color(colorValues[d+1], colorValues[d], colorValues[d+2]);
      strip.setPixelColor(i, c);
    }
    // update the strip
    strip.show();
  }
}
```

And there you have it. Simply program your Arduino with this sketch, plug it into your Raspberry Pi, then run the node script with `node app` and your LEDs should change color to the values specified (in the example above this was 1st LED RED, last LED blue, all others off).

Next time I'll be detailing how I setup the animation editor, and how we got the animations running on Node on the Pi using the Tween.js library. If you find this post useful, or are just feeling generous we're [fundraising for the Diodome](http://igg.me/at/diodome) project now.