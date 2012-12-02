---
title: Going Static
layout: post
tags: ['docpad','post','site','node.js']
date: 11/09/2012
---
After numerous Wordpress disasters, both of my own making and that of nefarious bots and spammers taking over the site, I've decided that its time to start over. I'm sure there will be further rants on the topic of the many faults (and virtues) of Wordpress at a later date, but for today I've had enough. This time I'm going with an alternative approach using [Node.js](http://www.nodejs.org) and [DocPad](https://github.com/bevry/docpad) to generate the site statically.

### Why static?

Static site generation is something that developers have been doing for years. Recently its had a resurgence thanks in part to [github pages](http://pages.github.com/) and the rise of popular site generators such as [Jekyll](https://github.com/mojombo/jekyll). Static site generation has a couple of advantages over something like Wordpress or other CMS systems. Firstly its really *really* easy to deploy your site, just upload it via FTP, GIT or whatever other mechanism you are comfortable with. There's no database setup required, no installation scripts to get correct its just plain old HTML files. **Nothing to go wrong**.

Secondly since its static, and the site is generated from simple text files, we can use all our standard development practices to work with our site. As good developers we all use version control (you do right? if not, [go here and get started](http://www.alistapart.com/articles/get-started-with-git/)). Version control (in my case using [Mercurial](http://mercurial.selenic.com/) or [GIT](http://git-scm.com/)) is part of our daily workflow and as such is a natural fit for developers maintaining a blog. You write a new post in a little text file, add it to your version control and upload it to your site. Job done, nothing to worry about. I already have offsite backups for my version control so the site is instantly backed up as part of that process. This sure as hell beats having to setup Wordpress database backups, and hoping that a mistake in a plugin or upgrade doesn't wipe the site.

### Getting started

To generate my site I have settled on [DocPad](https://github.com/bevry/docpad), but there's loads of [other generators out there](https://gist.github.com/2254924). I had originally considered using [Blacksmith](http://blog.nodejitsu.com/introducing-blacksmith) by nodejitsu, this seemed perfect as it uses simple HTML and CSS templates with markdown content files. Unfortunately however it doesn't currently work on Windows, so that ruled that out as Windows 7 (with Cygwin) is still my main development environment. Docpad works on everything, is a lot more flexible than Blacksmith and most other site generators as its not specifically focused one one specific type of site. It also features a good selection of [plugins](http://bevry.me/docpad/plugins) that allow you to use pretty much any templating, scripting or preprocessing language you care for. This combined with the livereload plugin allows us to develop our content and see live updates in our browser - a great feature especially for those not 100% sure on their markdown syntax.

Installation of DocPad is easy, first [install node.js](http://nodejs.org/), then [install docpad](http://bevry.me/learn/docpad-install) with the simple command:

```bash
	npm install -fg docpad@6.10
```
	
To get going quickly DocPad includes a helpful selection of skeleton sites I decided to go with the [Twitter Bootstrap skeleton](https://github.com/docpad/twitter-bootstrap.docpad):

```bash
	mkdir my-new-website
	cd my-new-website
	docpad run
```

Simply follow the instructions and you'll have a site running on http://localhost:9778 in moments.

DocPad has magically setup a local server running on node that can serve your site. There's a few advantages to this, for a start it means you don't have to get an Apache install working on your box just to test your site. It also means that DocPad can do some funky things like live reloading of your browser as your content is altered allowing you to quickly see the results of your changes. You can even deploy this entire stack very easily to a node hosting platform like [Heroku](http://www.heroku.com/) or [no.de](http://no.de/). I personally have decided to only host the statically generated files you will now find in your `out` folder, but I still use the local server for testing my content and changes before deploying my site to my web host.

In the next post I'll detail some more of the plugins available in DocPad and a little more about the changes I made to the default skeleton application to produce the site you see before you.