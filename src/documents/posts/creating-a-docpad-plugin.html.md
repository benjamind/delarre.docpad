---
title: Create a DocPad plugin
layout: post
tags: ['docpad','docpad-plugin', 'docpad-plugin-paged', 'site','node.js']
date: 12/02/2012
ignored: true
---
In order to complete the switch to DocPad I wanted my blog to support a paged index of my blog posts. I didn't want to display them all in one big long page, but split them up into groups of 3 or 5 with 'Older' and 'Newer' links to let users navigate the posts. Sadly this functionality isn't available by default in DocPad, but never fear DocPad is built with a plugin framework so we can add this functionality ourselves.

In the next couple of posts I'm going to walk through the creation of the [Paged Plugin](http://github.com/benjamind/docpad-plugin-paged) which I recently put together for just this purpose.

### Structure of a Docpad Plugin

