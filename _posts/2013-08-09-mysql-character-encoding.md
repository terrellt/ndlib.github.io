---
author:   jeremyf
category: practices
filename: 2013-08-09-mysql-character-encoding.md
layout:   post
tagline:  Its More of a Suggestion
title:    MySQL Character Encoding
tags:     ruby, rails, mysql, character encoding
---

In 2012 I was neck deep in character encoding.
What I was finding out was that our MySQL had been setup wrong.
And what took me a bit to find out was that MySQL character encoding was hell.

If it weren't for [BlueBox's Getting out of MySQL Character Set Hell](http://www.bluebox.net/about/blog/2009/07/mysql_encoding/) blog post, I think I would've been sunk.
This was some nasty stuff.

I ended up automating the steps to get me to mostly done.
I figured I'd end up sharing this code as it could be useful for others.
Your mileage may vary.

<script src="https://gist.github.com/jeremyf/6193878.js"></script>
