---
title: Creating a DocPad plugin
layout: post
tags: ['docpad','docpad-plugin', 'docpad-plugin-paged', 'site','node.js']
date: 12/02/2012
ignored: false
---
In order to complete the switch to DocPad I wanted my blog to support a paged index of my blog posts. I didn't want to display them all in one big long page, but split them up into groups of 3 or 5 with 'Older' and 'Newer' links to let users navigate the posts. Sadly this functionality isn't available by default in DocPad, but never fear DocPad is built with a plugin framework so we can add this functionality ourselves.

In the next couple of posts I'm going to walk through the creation of the [Paged Plugin](http://github.com/benjamind/docpad-plugin-paged) which I recently put together for just this purpose.

### A basic plugin

The docpad site has a [getting started guide](http://bevry.me/docpad/plugin-write) for how to setup a basic plugin thats a part of your site. Simply create the following folder and file structure where `yourPlugin` is the name of your plugin:

```bash
my-site/
	    src
	    ...
	    plugins/
	    		  yourPlugin/
	    		  			 yourPlugin.plugin.coffee
	    		  			 package.json
	    ...
	    docpad.coffee
	    README.md
	    package.json
```

The `package.json` is required and needs to use the format specified on the [docpad website](http://bevry.me/docpad/plugin-write).

Much to my dislike the standard way of writing plugins for DocPad is with CoffeeScript, you could do it with straight JavaScript but you won't get as much help from the documentation. I'll look into writing a straight JavaScript plugin guide sometime in the future, for now lets just use CoffeeScript.

### Pagination

For my paging implementation I wanted to be able to create a document in DocPad that was 'pageable', either looping over a collection of documents to render out sub-pages like on a blog index page, or looping a set number of times so that we can genereate pages on the fly or split up a long document across multiple pages. I tried out various ideas, and read [this issue](https://github.com/bevry/docpad/issues/116) which details some possible solutions for paging. In the end I settled on the following approach:

 * Set the `isPaged` property to true on pageable documents
 * Specify a `pageCount` property if you want a fixed number of pages **or**
 * Specify a `pagedCollection` property which specifies the name of a collection over which you want paging to be applied.
 * Specify a `pageSize` property to set how many documents are looped over in each page.

The plugin will function as follows:

 * Before we render the documents
 	* Loop over the documents
 	* Find those that have the `isPaged` property
 		* Read the `pageCount` or `pagedCollection` property to determine number of pages
 		* Read the `pageSize` property to determine document indexes in each page
 		* For every page we need to create:
 			* Clone the document and add it to a new collection
 			* Add a `page` object to the cloned document detailing current page details
 			* Add a `firstPageDoc` object to the cloned document which points back to the original document
 	* For every document in the new collection:
 		* Normalize and contextualize the new document
 		* Modify the `outFilename` and `basename` of the new document to reflect its page number
 	* Render the document collection

### To the code!
So lets get started. Here's the basic plugin code that we're going to start with:

```
 # Export Plugin
module.exports = (BasePlugin) ->
	# balUtil required for queing up tasks
	balUtil = require('bal-util')

    # Define Plugin
    class PagedPlugin extends BasePlugin
    	name: 'paged'

    	renderBefore: (opts,next) ->
    		docpad = @docpad

    		{collection,templateData} = opts

			pagesToRender = new docpad.FilesCollection()
...
```

This setups my basic plugin, it extends from `BasePlugin` and hooks into the `renderBefore` event, a full listing of all events in docpad is [available here](http://bevry.me/docpad/events). The `renderBefore` event receives two values in its `opts` argument, the `collection` and the `templateData`. The `collection` is the collection of documents that we are rendering, and the `templateData` is the data object that is passed to the documents when rendering. Note we also `require` the [`bal-util`](https://github.com/balupton/bal-util) module which we'll use later on.

Now we need to loop over our documents and figure out how / if they need paging:

```
...
			collection.forEach (document) ->
				meta = document.getMeta()

				if (!meta.get('isPaged'))
					return

				# let the page meta specify count or use 1 by default
				numberOfPages = meta.get('pageCount') or 1
				pageSize = meta.get('pageSize') or 5
				lastDoc = pageSize * numberOfPages

				# if pagedCollection is specified then use that to determine number of pages
				if meta.get('pagedCollection')
					pagedCollectionName = meta.get('pagedCollection')
					pagedCollection = docpad.getCollection(pagedCollectionName)
					numberOfPages = Math.ceil(pagedCollection.length / pageSize)
					lastDoc = pagedCollection.length

				# create a page object for this page
				document.set(page: {
					count: numberOfPages,
					number: 0,
					size: pageSize,
					startIdx: 0,
					endIdx: Math.min(pageSize,lastDoc)
				})

				document.set(firstPageDoc: document)

				# loop over the number of pages we have and generate a clone of this document for each
				if numberOfPages > 1
					for n in [1..numberOfPages-1]
						pagedDocData = document.toJSON()

						pagedDoc = docpad.createDocument(pagedDocData)
						pagedDoc.set(page: {
							count: numberOfPages,
							number: n,
							size: pageSize,
							startIdx: n*pageSize,
							endIdx: Math.min((n*pageSize) + pageSize,lastDoc)
						})
						pagedDoc.set(firstPageDoc: document)
						pagesToRender.add(pagedDoc)
...
```
So for each document in the collection we retrieve the meta object and test to see if the `isPaged` property is true. If it is we retrieve the `pageCount`, `pageSize` and optionally the `pagedCollection` properties. We use this information to figure out the number of pages we're going to render, and the start and end indexes of the documents in each page. If we've got more than one page then for each page we copy the current document, add a page object to the document and then add it to our collection of pages to render.

Almost there, now we just need to make these documents complete and contextual so they can be rendered:

```
...
			tasks = new balUtil.Group(next)

			pagesToRender.forEach (document) ->

				tasks.push (complete) ->
					document.normalize({}, complete)

				tasks.push (complete) ->
					document.contextualize({}, complete)

				tasks.push (complete) ->
					page = document.get('page')

					basename = document.get('basename')
					outFilename = document.get('outFilename')

					outFilename = outFilename.replace(basename,basename+'.' + page.number)
					basename = basename + '.' + page.number

					document.set('basename',basename)
					document.set('outFilename', outFilename)

					complete()

			tasks.push (complete) ->
				docpad.generateRender({collection: pagesToRender},complete)

			return tasks.async()
```
Here we've used the [`bal-util`](https://github.com/balupton/bal-util) module. This module has a few helpful little methods that make doing sequences of tasks a lot cleaner. Here we've created a sequence of tasks that will be executed asyncronously and after all tasks are completed the `next` method callback will be executed. The list of tasks is created by looping over the `pagesToRender` and for each document first normalizing it then contextualizing it then modifying its output and base filenames. Finally the last task added to the queue is to render the pagesToRender collection.

Normalization is basically the task which goes through the document and ensures all the values are updated appropriately, for instance if you change one value it might update another value as a knock-on effect. Contextualize is similar in that it sets the document up in the context of the rest of the site, for instance it generates the url property. So its important that we call both these methods on any new documents before we render them otherwise things just don't work quite right. This is also why we do the modification of the `basename` and `outFilename` properties after calling these methods as otherwise they would be reset back to the original document values.

By calling `tasks.async()` we execute the queue of tasks and then pass execution onto `next` allowing docpad to continue. With this the plugin is working! In the full version of the plugin I've gone ahead and added some helper functions to the `DocumentModel` that make it a bit cleaner to render pagination and operate over pages, if you want to take a look checkout the [full source here](https://github.com/benjamind/docpad-plugin-paged).

### Summary
So we've built a plugin that operates within our site to render out paged documents based off an original source document. I decided to share this plugin with everyone so I've packaged it up as a [package on NPM](https://npmjs.org/package/docpad-plugin-paged) if you want to use it in one of your own sites. In my next post I'll explain how I did that, and how to get Unit Testing working on DocPad plugins.