---
title: Unit Testing DocPad plugins
layout: post
tags: ['docpad','docpad-plugin', 'docpad-plugin-paged', 'site','node.js']
date: 12/03/2012
ignored: false
---

In the [last post](/posts/creating-a-docpad-plugin.html) I detailed how I put together the [Paged Plugin](http://github.com/benjamind/docpad-plugin-paged) for DocPad. After I got the plugin up and running on my own site, I decided I'd give back to the community and publish it on NPM for everyone to use. This entailed pulling the plugin out into a standalone module, and getting the unit testing working so I knew it was all good to go.

In order to do this I first had to setup the following folder structure:

```bash
docpad-plugin-paged/
					src/
						paged.plugin.coffee
						paged.tester.coffee
					test/
						 out-expected/
						 			  paged-count.html
						 			  paged-count.1.html
						 			  paged-count.2.html
						 src/
							 documents/
									   page-count.html.eco
							 paged.test.js
						 package.json
					Makefile
					README.md
					package.json
```

The `package.json` remains mostly the same as in previous exmaples with a small addition:

```
    ...

    "devDependencies": {
        "coffee-script": "1.4.x"
    },
    "scripts": {
        "test": "node ./test/paged.test.js"
    }

    ...
```

The `devDependencies` object specfies dependencies we have for development purposes only, in this case we need CoffeeScript to compile the plugin. The `scripts` object specifies the test runner that `npm` will use to test the plugin.

Note that I've added a `Makefile` to the folder, this is fairly standard for all DocPad plugins and I copied it from [one of the existing ones](https://github.com/docpad/docpad-plugin-partials/blob/master/Makefile).

The source for our plugin is now in the `src` folder in the `paged.plugin.coffee` file as before, I've also added some files here for testing purposes which I'll explain in more detail.

### Unit Testing

Since I'm releasing this code out into the wild it might also be wise to make sure it works so lets setup some unit testing. Thankfully DocPad has made this quite easy once you know how, with a suite of basic tests and expected output tests ready to roll so you don't have to do much to make sure your plugin is working correctly.

First we must create a 'tester' for our plugin, this is defined in the new `paged.tester.coffee` file:

```
# Export Plugin Tester
module.exports = (testers) ->
	# Define Plugin Tester
	class MyTester extends testers.RendererTester
		# Configuration
		docpadConfig:
			logLevel: 5
			enabledPlugins:
				'paged': true
				'eco': true
```

Notice here we're extending from the testers.RendererTester class. This is provided by DocPad and is designed to execute the plugin on a collection of documents and tests its output against a folder of expected results (`out-expected`). To figure out exactly how this works you might want to take a look at the source [available here](https://github.com/bevry/docpad/blob/master/src/lib/testers.coffee). But all you really need to know is the `docpadConfig` is used to configure an instance of DocPad to be used during testing, so here we enable our plugin and anything else we'll need in our tests.

Next we need to sort out our `test` folder which will actually contain and specify our tests. First up is our `package.json` file in the `test` folder, this is used to specify the dependencies for the test and to allow NPM to hook everything up for us:

```
{
	"name": "docpad-plugin-paged-testsuite",
	"version": "0.1.0",
	"private": true,
	"dependencies": {
		"docpad-plugin-eco": "latest"
	}
}
```

Now we can specify our actual test case in our `paged.test.js` file:

```
require('docpad').require('testers').test({pluginPath: __dirname+'/..',pluginName:'paged'});
```

All this does is get access to the testers module provided by DocPad and execute the test function passing in a configuration that tells DocPad how to run our plugin tests. All you actually have to specify is the `pluginPath` which in this case simply points to the parent folder which contains our plugins `package.json` and a `pluginName` which allows DocPad to properly instantiate our plugin. If you don't specify a `pluginName` property then DocPad will try and use the folder name of the plugin folder (in my case thats `docpad-plugin-paged` which does not match the filenames I used in my plugin).

Now for the content of the tests. Since we're using the RendererTester the test will consist of instanatiating a copy of DocPad and running it over the contents of the `/test/src/documents` folder and then comparing the resulting documents with the ones found in `/test/out-expected`. So simply place some documents in the folder and set it up as you would expect, for my simplest test I did the following in a file called `page-count.html.eco`:

```
---
title: 'Page Count Test'
isPaged: true
pageCount: 3
pageSize: 2
---
<header>Page <%= @document.page.number+1 %> of <%= @document.page.count %> : Documents [<% for i in [@document.page.startIdx...@document.page.endIdx]: %><%= i %><%= ',' if i < @document.page.endIdx-1 %><% end %>]</header>
```

This exercises my plugin by making it generate 3 pages each containing 2 documents, outputting a little HTML in each one. In order to make the test pass I just need to add the expected output to the appropriate folder each file goes a little somethind like this:

```
<header>Page 1 of 3 : Documents [0,1]</header>
```

See [the source](https://github.com/benjamind/docpad-plugin-paged/tree/master/test/out-expected) for more details.

### Running the tests
So we've setup the appropriate file structure, and added our tests. How do we actually execute them? Well its quite simple, but it requires a little more setup than you would hope at first. Because we added the `test` script to our `package.json` specifying a path to the test file we can use the command `npm test` to execute our test suite. However if you run that now you'll probably get an error message of some sort:

```
$ npm test

> docpad-plugin-paged@0.1.3 test C:\Workspace\docpad-plugin-paged
> node ./test/paged.test.js


module.js:340
    throw err;
          ^
Error: Cannot find module 'docpad'
    at Function.Module._resolveFilename (module.js:338:15)
    at Function.Module._load (module.js:280:25)
    at Module.require (module.js:362:17)
    at require (module.js:378:17)
    at Object.<anonymous> (C:\Workspace\docpad-plugin-paged\test\paged.test.js:1:63)
    at Module._compile (module.js:449:26)
    at Object.Module._extensions..js (module.js:467:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Module.runMain (module.js:492:10)
npm ERR! Test failed.  See above for more details.
npm
```

Not good. In order to get this working I had to turn to [Benjamin Lupton](http://balupton.com/) the creator of DocPad for some answers. In short to setup the testing structure we need to get the development version of DocPad, compile it, and setup symlinks so that your plugin tests can use it. So to get everything setup you'll need to install the DocPad source, and setup a symlink to it, thankfully this is quite easy:

1. `git clone https://github.com/bevry/docpad.git` - Clone the DocPad respository into your workspace (not your plugin folder)
2. `cd docpad` - move into the new folder
3. `npm install` - install the DocPad dependencies locally
4. `npm link` - link the DocPad folder you have now created into npm, this means when you use DocPad on your machine it points to this working folder
5. `make compile` - compile DocPad so its ready to use

Now we need to do something similar for your plugin:

1. `cd docpad-plugin-paged` - move into the folder for your plugin
2. `npm install` - install the dependencies for your plugin
3. `npm link docpad` - add the linked version of DocPad to your plugins dependencies
4. `make compile` - compile your plugin
5. `make test` - run the tests!

Hopefully your tests should now run and pass!

### Publishing on NPM

To round everything out I simply added some details to my `README.md` file and added an `.npmignore` file to my folder containing the following:

```
.travis*
Makefile

src/
out/test/
test/
```

This just tells npm to ignore my test folders and other things that are unnecessary. I then setup my user account on https://npmjs.org/signup and added my user account details to my npm installation with `npm adduser`. After that publishing to npm is accomplished with the `npm publish` command.

There you have it, a fully tested, published plugin for DocPad. Hopefully this will be the first of many, I've a few other ideas for things I'd like to see in the project so we'll see if I can find time to make them. If you are trying to get a DocPad plugin up and running and are having problems give me a shout, or drop into talk to the incredibly helpful DocPad guys on the irc.freenode.org server channel #docpad.
