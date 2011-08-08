---
layout: post
name: blog-migration-to-github.
layout: post
title: Migrated from blogger to github pages
time: 2011-08-08 16:55:00 +10:00
---


A while back when I was setting up the project site for [jentrata.org](http://jentrata.org) I started looking around for site hosting options and found [github pages](http://pages.github.com/) and since jentrata was already using github for source code hosting it made sense to use it (and it seems to be what all the cool kids are using).

In my research of github pages I found a few blog posts on migrating which were helpful

* [Moved Blog to Jekyll and GitHub Pages](http://www.alexrothenberg.com/2011/01/27/moved-blog-to-jekyll-and-github-pages.html)

* [List of sites using github pages](https://github.com/mojombo/jekyll/wiki/Sites)

I am kind of digging [jekyll](https://github.com/mojombo/jekyll) the scripting language behind github pages it's pretty simple but quite powerful but a lot of the extensions don't work with github. Which I guess is understandable since it lets you run arbitrary ruby code.

I really hated the blogger interface and it made a good excuse not to blog :). Also I like the idea of treating my blog like code.

	git clone git@github.com:aaronwalker/aaronwalker.github.com.git
	cd aaronwalker.github.com.git
	vim _posts/yyyy-mm-dd-new-post.md
	write some random crap that probably nobody will read :)
	git add _posts/yyyy-mm-dd-new-post.md
	git commit -m 'new post'
	git push origin master
	done

I think this lowers the barrier for me...

I'll put together a more technical post on how I did the migration....

Let git it on!!!! (oh god that's bad, so very bad)

