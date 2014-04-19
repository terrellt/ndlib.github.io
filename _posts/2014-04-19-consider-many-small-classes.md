---
author:   jeremyf
category: practices
filename: 2014-04-19-consider-many-small-classes.md
layout:   post
tagline:  Like on Moving Day, Its Easier to Move Many Small Things
title:    Consider Many Small Classes
tags:     ruby
---

When I first started my professional software development, I was working in a system that had a maximum of 8 characters for variable names.
This meant all of the database columns, table names, program names, were a garbled mess.
Those 8 character names sure were fast to create and type.

In design conversations the team would often refer to the 8 character handle instead of the more meaningful description.
"Yeah, lets add a new column to PRPRSCVG so that PRR104E can run the new calculations."
It always reminded me of Robin William's line in Good Morning Vietnam.

These short handles were fast if you were familiar with the code.
But for those who just visiting, the code was packed with dense jargon.

One of our system architects began encouraging us to refer to those tables and programs by their descriptive names.
It worked, to a point, but we still fell back on these fast handles for communicating with the domain; And left new people to falter.

When I first started my professional web development career, I was in heaven.

Variable names, classes, tables, and other symbols could be of any length.
I was able to express what these variables meant instead of relying on comments that may or may not have been accurate.
While longer variables were both slower to type and name, they were much more expressive.
And it became both easier to talk about the code as well as maintain the code.

I had also started developing in Ruby and loved its use of faux multiple inheritance via module mixins.
With these modules I began thinking in greater terms of reusability.

I would implement a publishing workflow for a WebPage.
Then I would be asked to implement something similar for a NewsArticle.
With a little bit of work, I ended up extracting a mixin module for WebPage and NewsArticle.

Both a WebPage and NewsArticle could mixin a Publishable module, adding common methods to both of those Nevermind the reality that the Publishable module required intimate knowledge of WebPage and NewsArticle, I was coding fast!

And then along comes Alec.
He took over maintenance of applications that I was leaving behind.
He's done it twice and both times with a minimal amount of public cursing.

Those mixin modules were a liability for new developers.
The modules may look self-contained, but in reality weren't.

The module mixins were creating objects with a large number of public methods.
Which makes it easy to operate directly on those objects.
And soon I had a big ball of mud.

I should take a cue from Corey Haines:

>> "I use modules in this way as a step in the path towards a better design.
>> Separating out aspects of a class into modules can help find hidden dependencies, as well as highlight all the different responsibilities a class has.
>> But they are rarely the place to stop." -- Understanding the Four Rules of Simple Design

My interpretation of this is that modules are a valid stepping stone for refactoring.
But they should not be the final stop.
