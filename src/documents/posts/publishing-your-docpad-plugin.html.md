---
title: Unit Testing DocPad plugins
layout: post
tags: ['docpad','docpad-plugin', 'docpad-plugin-paged', 'site','node.js']
date: 12/04/2012
ignored: true
---

In the last couple of posts I've detailed how I put together the [Paged Plugin](http://github.com/benjamind/docpad-plugin-paged) for DocPad. After I got the plugin up and running on my own site, I decided I'd give back to the community and publish it on NPM for everyone to use. This entailed pulling the plugin out into a standalone module, and getting the unit testing working so I knew it was all good to go.

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

The package.json remains mostly the same as in previous exmaples with a small addition:

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

### Unit Testing

Since you're releasing this code out into the wild it might also be wise to make sure it works so lets setup some unit testing. Thankfully DocPad has made this quite easy once you know how, with a suite of basic tests and expected output tests ready to roll so you don't have to do much to make sure your plugin is working correctly.
