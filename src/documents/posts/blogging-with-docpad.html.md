---
title: Blogging with DocPad
layout: post
tags: ['docpad','site','node.js']
date: 11/10/2012
---
In the last post I explained a little about why I've moved this site away from Wordpress to a static generator approach using [DocPad](https://github.com/bevry/docpad) where I generate this blog from a set of files rather than from a database. This time I'm going to detail a little bit of how DocPad works and the changes I've made to get this site running as you see it now.

The [Twitter Bootstrap skeleton](https://github.com/docpad/twitter-bootstrap.docpad) I'm working with provides a basic set of DocPad plugins to start with. So I'm going to write a little bit about each one and how it fits into the DocPad system. DocPad is pretty smart in that it lets you use lots of different markups and languages. Through its plugin architecture it automatically runs the appropriate scripts based on the file extensions of the files in your project. So for instance if we have a file named `hello.html.md` it will automatically process any handlers for the `.md` extension and then expect the result of that to be a `.html` file. You can even feed one language into another and so on using this extension based mechanism. So a file called `hello.html.md.eco` will process the file as an Eco file first, then as a Markdown file before outputting it to HTML.

###Marked
The [Marked Plugin](https://github.com/docpad/docpad-plugin-marked/) converts files containing text in Markdown format to HTML files. Others far more qualified than I have espoused the [many benefits of Markdown](http://www.codinghorror.com/blog/2008/05/is-html-a-humane-markup-language.html) so I'm not going to dive into that very much. Our main use for Markdown in this case is for the content of the site itself. All the text you are reading now is written in a file called `blogging-with-docpad.html.md` which is processed by DocPad into the HTML your browser has just read.

###Eco
This skeleton also uses [Eco](https://github.com/sstephenson/eco) which was new to me and might be to you too if you haven't been keeping up with the developments in [CoffeeScript](http://coffeescript.org/).

Eco is a form of CoffeeScript designed specifically to make it useful for creating templates. It supports various useful syntactical sugar allowing you to easily evaluate expressions, output values and properties and keep them all appropriately escaped.

```coffeescript
	<ul class="nav nav-pills">
		<% for document in @getCollection('pages').toJSON(): %>
			<li typeof="sioc:Page" about="<%= document.url %>" class="<%= 'active'  if @document.url is document.url %>">
				<a href="<%= document.url %>" property="dc:title"><%= document.title %></a>
			</li>
		<% end %>
	</ul>
```

This little snippet is used to create the navigation links across the top of this site. It looks over all the documents found in the 'pages' collection (which is just all the documents found in the `documents/pages` folder) and outputs an `li` element for each of them containing the appropriate data and markup. I've not used CoffeeScript or Eco much, but I can say that it seems like a powerful little setup that makes templating pretty easy and fast. Its also nice not to have to deal with too much of the JavaScript syntax when writing code primarily dealing with markup as it can often get a little messy.

###LiveReload
The [LiveReload Plugin](http://docpad.org/plugin/livereload) in my opinion really makes the DocPad experience great. Simply adding this to your site makes it instantly reload whenever DocPad detects that a file has been changed. This means I can edit my content in the text editor of my choice, hit save and instantly see the output in my browser. Without this I think the whole statically generated site approach would become cumbersome pretty quickly.

###CleanUrls
The [Clean Urls Plugin](https://github.com/docpad/docpad-plugin-cleanurls/) simply removes the .html from urls when browsing the site through the docpad server. I personally have removed this from my project since I know I'm going to output this site only to static html files so I would like my development server to behave just like the static files would.

###Stylus
The [Stylus Plugin](https://github.com/docpad/docpad-plugin-stylus/) allows you to use the [Stylus](http://learnboost.github.com/stylus/) language to produce your CSS files. This allows for simpler styling specifications which makes your CSS a little easier to manage. You could also use the LESS plugin to achieve a similar thing, both are installed by default in this skeleton.

###HighlightJS
With the above plugins you can get a pretty decent site going very quickly, however since my site is primarily about coding I have decided I wanted to use the [HighlightJS Plugin](https://github.com/docpad/docpad-plugin-highlightjs/) also. This I installed like so:

```bash
	npm install --save docpad-plugin-highlightjs
```

Now that highlightjs is part of the build process of the documents in my site, I just need to add code blocks using syntax similar to the following, here's the code block for the above snippet of bash script.

```
	```bash
		npm install --save docpad-plugin-highlightjs
    ``````

Here we're using the backtick (`) character to let us specify the language of the code block. Normal code blocks in Markdown are just one tab indented, but since we want to give this code block a specific language for highlighting we use the backtick syntax above. Finally to get the syntax highlighting to actually higlight your code you will need to add some CSS to your site to add the appropriate colorization. I like the GitHub style so I've included the github.css in my styles folder from the original [HighlightJS project here](https://github.com/isagalaev/highlight.js/tree/master/src/styles).

###Summary
The Twitter Bootstrap skeleton has got us up and running remarkably quickly. I really like the fact that I now have a really fast and easy way to create this blog, and that it all fits in with my normal development work flow. No longer do I have to mess around with Wordpress malware, or worry about database backups.

I still haven't dug into the templating capabilities too much yet, for instance my blog right now is just a list of articles. I would quite like to break them up into summary sections and full articles with a `more...` link. I'd also like related article links to be automatically generated from post tags. These are all things to look into for my next post.