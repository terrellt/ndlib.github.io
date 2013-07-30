---
author:   jeremyf
category: practices
filename: 2013-07-29-starting-a-new-rails-project-with-tests.md
layout:   post
tagline:  Start Doing What You Should Be Doing
title:    Starting a New Rails Project with Tests
tags:     ruby, rails, testing, hydra, rspec
---

## Introduction

This post is to help you get started writing your first test for a Hydra powered Rails project.
It is generally applicable for writing your first integration test for a Rails project.

* [My First Rails Project](#my-first-rails-project)
* [Types of Tests](#types-of-tests)
* [Write Your First Rails Test](#write-your-first-rails-test)
* [But My Project is Already Started](#but-my-project-is-already-started)

<blockquote>
We write tests first because we find that it helps us write better code.
Writing a test first forces us to clarify our intentions, and we don’t start the next piece of work until we have an unambiguous description of what it should do.
</blockquote>

By Kent Beck in the forward for "Growing Object-Oriented Software, Guided by Tests" by Steve Freeman and Nat Pryce.

## My First Rails Project
<span id="my-first-rails-project"></span>

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
<span id="types-of-tests"></span>

As you are exploring testing you may hear about all sorts of tests: Unit, Functional, Integration, Component, System, Exploratory, Feature, Acceptance, etc.

You may hear about Test-Driven Development and Behavior-Driven Development.

There is a lot of discussion regarding tests.
At this point, I would recommend thinking about tests as belonging to two categories:

* Isolated
* Integrated

Alan Kay, an author of Smalltalk, coined the term object-oriented, positing that objects are similar to biological cells that send each other messages.

Think of isolated tests as testing a single cell.
This is where you make sure the cell is operating in a well-structured manner.
*Isolated tests help me write code the right way.*

Think of integrated tests as tests that coordinate the interaction of multiple cells.
This is where you make sure your organism is operating as expected.
*Integrated tests help me write code that is doing the right thing.*

I would highly recommend "The Clean Coder: A Code of Conduct for Professional Programmers" by Robert C. Martin. In chapter 8 he explains the various tests, as well as the ratio of each type of test.

## Write Your First Rails Test
<span id="write-your-first-rails-test"></span>

As a contributor to [Hydra projects](http://projecthydra.org), I am aware that there new Rails developers joining the project.
I wanted to provide a "for developers" walk through for creating a new Hydra project, with testing at the forefront.

So I wrote a [Hydra Capybara Walkthrough](https://github.com/jeremyf/hydra-capybara-walkthrough).
It would be helfpul to read through the [commit log in reverse order (earliest to latest)](https://github.com/jeremyf/hydra-capybara-walkthrough/blob/master/COMMIT-LOG-REVERSE.md).

Of particular interest are:

* [Adding spec and comments for testing catalog [f7d44ae]](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/f7d44ae1bbe2fbb62e72b7f9831cd64ad8ad5fe7). This is my comments for the first test I want to write.
* [Writing executable tests for capybara tests [1c3b445]](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/1c3b445c3f77e8a87b3675d2539ee0770853f017). Converting the comments into functional ruby code.
* [Adding Thing model and corresponding spec [f7492b9]](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/f7492b95ad33a8f0e9904e441f35e76851ae492e). Using an ActiveFedora generator for a model. I get a free test.
* [Adding a datastream and title to Thing [4c366ff]](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/4c366ff891ba35ee7c74fa131c31554f742479cb). Making the Thing do something.
* Refactoring spec to consolidate behavior [dcc4423](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/dcc4423b5877c7f319eeeff0d8d9c25fcc77ce20). Don't forget that refactoring your tests is just as important.
* [Adding persistence verification for Thing [bf9cf0f]](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/bf9cf0fee1192527df3afaa3c704e9d4d1bed101). Think about writing custom RSpec matchers. By writing a custom matcher, you are encoding what a potentially large chunk of test code means. And it becomes reusable.
* [Adding end to end test of creation [5f8a30b]](https://github.com/jeremyf/hydra-capybara-walkthrough/commit/5f8a30b778197b9aba48eb52ff16434ba1142dbd). Verify that the application is behaving by writing a straight forward Capybara spec.

### A Word of Caution

By their very nature, Capybara tests are extremely fragile.
You are testing your UI, an extremely volitale windown into your application;
Your Capybara tests will creak, groan, and very likely break with alarming frequency.
But they will be helpful.
Just keep the number and scope of integrated tests well groomed.

## But My Project is Already Started
<span id="but-my-project-is-already-started"></span>

[Jon Hoyt](http://twitter.com/jonmagic), a good friend of mine, and a tenacious, professional developer once told me about a project he joined.
The project had no tests.
And the code was in a disarray.
He said that he wrote up 6 integration tests that covered, what he felt to be, the vast majority of the code-base.

With those six tests he worked his way through the code:

* Refactoring towards cleaner code
* Writing unit tests to improve his understanding of the code
* Adding new functionality

He told me that this was a more painful method than starting with tests first, but he didn't have that luxury.
He said that he leaned heavily on those tests, relying on them to indicate the general health.
When his initial 6 integration tests didn't catch a bug, he'd write a component or unit test that would.

I would recommend writing up at least a handful of Integration tests for your project that represent the mission critical behavior of your application:

* Login as a user, upload a file, and make sure they can find the file in their dashboard
* An anonymous user searches for publicly available files, clicking on one, then viewing the details and finally downloading the file.

We have the tools to automate those tests, so please do so.
That way, as the eco-system changes beneath you, or you are asked to add new functionality, you have a canary to warn you if you are heading down a dangerous tunnel.