---
author:   jeremyf
category: practices
filename: 2015-04-30-answers-to-questions-from-the-computer-science-curious.md
layout:   post
tagline:  Its Not What You've Learned Its How You Learn
title:    Answers to Questions from the Computer Science Curious
tags:     ruby, rails, design, coherent-development, novice
---

**What follows is my email response to someone curious about expanding their computer science background.**

> Since you have a CS background, what did you find valuable about it? Do you find that what you learned applies a lot to your current work, or was it just a bunch of abstract math out in space that you promptly forgot? If you were in my position and your goal was to become the best developer possible and do the most good (and not necessarily to get any specific degrees, although it would be nice), how would you go about doing it?

A bit of my background. My degree is in Computer Science and Applied Mathematics, but the devil is in the details. I went to a small liberal arts school (goshen.edu). When I started my undergraduate studies I was a math major with an interest in a computer science minor. I had done programming as a kid but I wouldn't have considered myself a hacker.

My first two years I took 1 CS class; Intro to Programming. Prior to this year I would've said it was worthless; Having mentored a few people in Rails bootcamps I see there was an important lesson that was drilled into my skull: variable scope.

At the beginning of my Junior year I took Data Communications (worthless). I took Data Structures which was somewhat worthless. We spent too much time in theory and not enough time working through stuff. We learned to implement an array and stack via linked lists. We talked about performance and algorithmic speeds. If anything they helped me be mindful of those challenges or potential pitfalls. The data structure classes provided guidance on organization; But no more than a solid English class or Math class or DMing a D&D campaign did.

I also took a SQL class (in MS Access). It was helpful to learn about data normalization; That has stuck with me, but really my pedantic condescending DBA at my first job taught me more.

My Senior year they offered a new major: Computer Science and Applied Mathematics. To achieve that required credits equal to one major and one minor however the credits could come from either discipline. Being 3 credits away from a minor in CS, I grabbed that degree and said "No take backs".

My "formal" training ended there. I am a patterns guy; I see abstractions and noodle on them. I kick them around for awhile until I can actualize them. My first job was in insurance, and I saw what I still consider to be the high water mark of software design. It was a database driven suite of applications for calculating insurance rates; We built our math formulas in the database by articulating something along the following lines: Previous step, Current Step Operand, Current Step Function.

I was neck deep in what I would later discover to be functional programming. It was powerful and amazing. Something complicated but very rules based. I spent the next portion of my career focusing on querying; Writing oodles of SQL and modeling data for reporting and computation.

Then I entered web development; I brought with my a strong DB background as we worked on several database applications that had 100+ tables each, relying on normalized data and optimized queries to ensure a solid user experience. During this time I was voraciously reading all kinds of software books.

* The Pragmatic Programmer by Andy Hunt and Dave Thomas
* Ruby Pickaxe Book
* Design Patterns in Ruby by Russ Olsen
* Smalltalk Best Practice Patterns by Kent Beck
* Confident Ruby by Avdi Grimm
* Much Ado About Naught by Avdi Grim
* Martin Fowler's Refactoring
* Robert Martin's Clean Code
* Head First Design Patterns in Java
* Robert Martin's Clean Coder
* Corey Haines's "4 Rules of Simple Design" https://leanpub.com/4rulesofsimpledesign

The second to last slide of this presentation contains links to most (and more) https://speakerdeck.com/jeremyf/taming-hydra-and-rails

> Since you have a CS background, what did you find valuable about it?

I found little of my background to be valuable by itself. Instead I found most helpful the practice of looking for patterns and being ever vigilant on repetition of knowledge (DRY principles).

If I were to take a class, I'd look at an advanced SQL class. Schemas are powerful…but not in the way I often see them used. Articulate rules via schemas; Don't fuss over the shape of documents, those are messy in library world. I'd also consider something in Graph theory.

> Do you find that what you learned applies a lot to your current work, or was it just a bunch of abstract math out in space that you promptly forgot?

I took some graph theory classes, and wish that I could recall more of it. However, most of the classes taught me something at the meta-level:

* Always show your work; Favor explicit over implicit.
* Be precise in language and communication.
* Articulate assumptions and why you are doing things. Don't document what you are doing; that's your code and tests. Instead document why you are doing something.

The facts are long gone, but the discipline is something I carry with me.

> If you were in my position – Masters in library science last spring, and before grad school I had never done any programming or even considered it as a viable career path - and your goal was to become the best developer possible and do the most good (and not necessarily to get any specific degrees, although it would be nice), how would you go about doing it?

To be the best developer possible, practice, practice, practice. Abhor repetition; Seek to automate it. For example I wrote scripts:

* Download and take a snapshot of my emails to store for reference
* Generation of some role-playing procedures (dice rolling, graph generation, map generation and XML transformation)
* Pre-Deployment procedures to get our application ready for deploy (instead of writing documentation write the script that does it, and keep using it)
* Transform XML documents into a YAML payload to populate a static site for a game convention I worked on.

Ultimately, I've written tools that relate directly to a passion of mine.

Use source control (git is my favorite tool); Make small commits with meaningful messages. Commit early and often. Branch out with new ideas. Explore. Delete that code and start again.

Read other people's code, use other people's code, contribute to other people's code (or documentation). Read books on how to execute the craft, not the details of how to solve an algorithm problem.

Before you start down trying to write the code solution to a problem, make sure you understand how you will verify that your solution works. One means is writing an automated test to verify this; Another is articulating the expected output of the given input and processing.

Get comfortable with a test framework; Practice using a test framework as you solve a known problem (See the Bowling Kata). If you attempt to learn a testing framework and try to solve a new problem at the same time, you now have two points of failure.

Spend some time working on Project Euler challenges and Code Wars; See how others are solving problems.

Fall in love with your code editor; Learn its ins and outs: shortcuts, macros, snippets, refactoring options, CTags, etc. Spend time sharpening your coding tools.