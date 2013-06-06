---
author:   jeremyf
category: practices
filename: 2013-06-06-starting-a-new-rails-project-with-tests.md
layout:   post
tagline:  Start Doing What You Should Be Doing
title:    Starting a New Rails Project with Tests
tags:     ruby, rails, testing, hydra, rspec
---

## Introduction

This post is to help you get started writing your first test for a Hydra powered Rails project.

* My First Rails Project
* Types of Tests
* Write Your First Rails Test

<blockquote>
We write tests first because we find that it helps us write better code.
Writing a test first forces us to clarify our intentions, and we don’t start the next piece of work until we have an unambiguous description of what it should do.
</blockquote>

By Kent Beck in the forward for "Growing Object-Oriented Software, Guided by Tests" by Steve Freeman and Nat Pryce.

## My First Rails Project

My first Rails project started December 2005.
I dove in quickly and immediately began writing the production code.
I was learning Rails for the first time.
It was a volitale period in which Rails and its echo system were rapidly evolving.

So much was changing, and I was working at learning it all, that I didn't feel I could add "writing tests" to the list of things I could do.

And I quickly learned this was the path of pain.
Not only was I writing bugs – every developer does this – but the ecosystem was changing from beneath me.
Everything was in motion and my code was sloshing around, collapsing in on itself.

It was a nightmare of fragility.

## Types of Tests

As you are exploring testing you may hear about all sorts of tests: Unit, Functional, Integration, Component, System, Exploratory, Feature, Acceptance, etc.

You may hear about Test-Driven Development and Behavior-Driven Development.

There is a lot of discussion regarding tests.
At this point, I would recommend thinking about tests as belonging to two categories:

* Isolated
* Integrated

Alan Kay, an author of Smalltalk, coined the term object-oriented, positing that objects are similar to biological cells that send each other messages.

Think of isolated tests as testing a single cell.
This is where you make sure the cell is operating in a well-structured manner.
*Isolated tests ensure that you are doing it the right way.*

Think of integrated tests as tests that coordinate the interaction of multiple cells.
This is where you make sure your organism is operating as expected.
*Integrated tests ensure that what you are doing is right.*

I would highly recommend "The Clean Coder: A Code of Conduct for Professional Programmers" by Robert C. Martin. In chapter 8 he explains the various tests, as well as the ratio of each type of test.

## Write Your First Rails Test

Install Hydra

Download Jetty

Wire up Rspec

Wire Up Capybara

Write Your First Test

Immediately get Capybara going
  - test the catalog controller
  - a word of caution, Capybara tests are fragile
