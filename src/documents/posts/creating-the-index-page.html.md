---
title: Creating the index page
layout: post
tags: ['docpad','post','site','node.js']
date: 11/12/2012
---
Now that we've got Docpad up and running, the last thing thats absolutely essential is to get the site showing the blog posts on the index page. You'd think this would be simple, and it really is, but it stumped me for a little while before I found the appropriate documentation.

Now I'm not certain this is the best way of doing things but it works for me and its simple enough. To display the last N blog posts on your index page do something like this:

```coffeescript
	<% for document,i in @getCollection('posts').toJSON(): %>

	<% break if i > 2 %>

	<article id="post" class="post">
		<h1><a href='<%=document.url%>'><%= document.title %></a></h1>
		<div class="post-date"><%= document.date.toLocaleDateString() %></div>
		<div class="post-content">
			<%- document.contentRenderedWithoutLayouts %>
		</div>
	</article>
	<% end %>
```

This simply says loop over the collection of 'posts', break the loop if the `i` variable is greater than 2 (remember this is a for loop so its zero indexed). Then we simply render the article, and use the `document.contentRenderedWithoutLayouts` rather than the usual @content. We do this since we're already 'laid out' so we don't need the full layout of the document again.