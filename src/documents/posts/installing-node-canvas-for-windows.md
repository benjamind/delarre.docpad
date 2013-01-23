---
title: Installing node-canvas for Windows
layout: post
tags: ['node','node-canvas','node.js']
date: 01/23/2013
---
I've recently been experimenting with [node.js](http://nodejs.org) for a possible new side project. So far my experience has been great, there's a lot of fun to be had working with node and there's a massive raft of libraries available to make your life really easy.

### CSS Awesome
I wanted to use [Stylus](http://learnboost.github.com/stylus/) in my application to compile and normalize my CSS. I cannot overstate just how great using Stylus as part of your workflow for CSS really is, its saving me loads of time and my CSS feels clean and tidy. Anyway, one addition you can make to Stylus to further improve its awesomeness is to add [Nib](http://visionmedia.github.com/nib/) which allows for really cool stuff like cross-browser CSS3 extensions, various short-hands that make life easy, and most of all automatic generation of Gradient images for IE and older browsers that don't support gradients.

However, all this awesomeness comes with a price, you must install [node-canvas](https://github.com/learnboost/node-canvas) into your app so that it can render these gradient images on the server side. While node is proving to be largely platform agnostic, some modules such as this one require native code compilation. While this is usually pretty straightforward on *nix based operating systems its a bit of a headache on Windows. I've now had to get this running on a couple of different boxes and keep forgetting where to get all the relevant pieces, so I thought it was time for a blog post so I have this stuff easily to hand and maybe it will help some of you out too.

There's an [article here](https://github.com/LearnBoost/node-canvas/wiki/Installation---Windows) that documents how to install node-canvas on Windows, but I found its missing a few details, so I've outlined these below.

### node-gyp setup
Native compilation on node is achieved with the node-gyp module. This tidies up all the little bits of pain we usually associate with compiling things on various platforms. In order for it to work on Windows we'll need to install some of its dependencies which are [listed here](https://github.com/TooTallNate/node-gyp#installation).

1.	Install [Python 2.7.3](http://www.python.org/download/releases/2.7.3/#download)
	_only 2.7.3_ nothing else works quite right!

	If you use Cygwin (as I do) ensure you _don't_ have Python installed in Cygwin setup as there will be some confusion about what version to use.

2.	Install [Visual Studio C++ 2010 Express](http://go.microsoft.com/?linkid=9709949)

3.	(64-bit only) Install [Windows 7 64-bit SDK](http://www.microsoft.com/en-us/download/details.aspx?id=8279)

	If like me, the SDK fails to install with the following error:

	*Installation of the “Microsoft Windows SDKfor Windows 7” product has reported the following error: Please refer to Samples\Setup\HTML\ConfigDetails.htm document for further information.*

	Then you should follow the [instructions here](http://support.microsoft.com/kb/2717426) to get the installation to work again.

### node-canvas installation
The node-canvas module uses GTK to do its rendering on the server side, so you'll need to install a copy of the GTK binaries on your machine. You'll want the 'All in one' binary package:

* (32-bit) - http://ftp.gnome.org/pub/gnome/binaries/win32/gtk+/2.24/gtk+-bundle_2.24.10-20120208_win32.zip
* (64-bit) - http://ftp.gnome.org/pub/gnome/binaries/win64/gtk+/2.22/gtk+-bundle_2.22.1-20101229_win64.zip

Download the appropriate package, and unzip it to the `C:\GTK` folder, any other folder and you'll have to make configuration changes to node-gyp so its probably best to just put the library there.

Now that you have node-gyp setup properly you can install node-canvas like you would any other module, either globally with:

	npm install -g canvas

Or locally in your application using a dependency in your ``package.json` file.

Now hopefully your canvas installation will work perfectly from here on out. However, you might still encounter errors. If you do, you could try one last nasty hack that really isn't ideal but did solve some final dependency issues for me. It appears that on some installations the `.dll` files for GTK do not get found by the native code running for node-canvas. In order to make this work simply copy all the dll files from the `C:\GTK\bin` folder to your `node_modules/canvas/build/Release` folder. That should resolve any niggling dependency issues.

### Node + Windows
While I beginning to love developing with Node, I am really hating the niggling issues it still has with Windows. While I could switch to OS X or a flavour of Linux, I'm really disinclined to at this stage as the Node issues are the only thing that really grates in my development process right now and I'm very proficient with my current choice of OS.

Perhaps things will improve, I'm certainly going to try and stick with it for now and where possible contribute back to the Node community to get things more polished.