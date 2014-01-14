---
author:   jeremyf
category: practices
filename: 2014-01-14-a-quick-reference-to-refactoring.md
layout:   post
tagline:  Drifting Towards Many Moving Parts
title:    A Quick Reference to Refactoring
tags:     ruby
---

Rails provides a rather powerful [polymorphic_path](http://api.rubyonrails.org/classes/ActionDispatch/Routing/PolymorphicRoutes.html#method-i-polymorphic_path) method in the views.
When working with a heterogenous collection of objects in the view, you can use polymorphic_path to generate a URL to the correct resource (as defined as a resource in your routes).
It introduces some potentially gnarly issues, especially if you create generalized controllers.

The [recent refactoring blog post at Thoughtbot](http://robots.thoughtbot.com/code-show-and-tell-polymorphic-finder) shows the before and after state of the code.
The draw attention to various patterns that were used as well as the pros and cons of the refactor.
In particular, I find the following quote very emblematic of both what was done as well as where things need to go in Curate.

> In summary, using the new code is easier, but understanding the details may be harder.
> Although each piece is less complex, the big picture is more complex.

Please take some time to go and read the post. And while you are at it, add [Thoughtbot's blog](http://robots.thoughtbot.com) to your RSS Reader.