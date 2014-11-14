---
author:   jeremyf
category: practices
filename: 2015-10-07-code-is-a-conversation.md
layout:   post
tagline:  Coherent Development (Part 2)
title:    Code is a Conversation
tags:     rails, design, projecthydra, coherent-development
---

*Part of an ongoing series for [Coherent Development](/practices/coherent-development-part-1/)*

As a software developer, it is my responsibility to take imperfect requirements
and translate them into a set of instructions for a computer to perfectly execute.

And there in lies the tension. I am always playing a [game of telephone](http://en.wikipedia.org/wiki/Chinese_whispers) with numerous players:

* Product owners
* Other developers
* Me
* Any frameworks I may be using
* The code interpreter

And to further complicate the game, there is the past, present, and future versions of each of those players.

From experience past versions of the players, especially myself, do their best to mess things up for present and future players; Not because they are malicious, but because they don't have the wisdom and context of present and future players.

As a developer, what can be done about this?

## Express Intent

First make sure the code you write expresses intent.
This is challenging.
You wrote it today, so present you understands it.
Instead seek the help of others.

I like to think in terms of an author and editor relationship.
The editor is there to help you communicate your message to a larger audience.
You can self-edit your work.
But through the self-editing process you are communicating with yourself.

I recommend creating a Pull Request and asking for someone to help review it.
If a Pull Request is not feasible, have another developer take a look at your changes.

The Pull Request should be composed of the following:

* A well crafted commit message
* Code changes (this is a given)
* Automated tests
* Optional: comments explaining any tricks or gotchas in the code

## Commit Messages

I cannot emphasize enough that a well written commit message is valuable.
It travels along with your code.
It provides context to future developers regarding the conversation you are having right now.

It is okay to write disposable commit requests.
But once you are ready to have your code join the ongoing conversation, take off your developer and put on your communication hat.

Help out your future self by writing something meaningful.
Help out your colleague that might be reviewing your code.

Give context to the conversation.
Try to help future you not hate present you.

Consider the following commit message:

> Better fix for moving works from embargo to public
>
> Undo previous changes to app/repository_models/curation_concern/embargoable.rb
> and implement a simpler and better fix for users not being able to
> move a work from embargoed status to open/public.  The problem is
> that the setter method for embargo_release_date= gets called twice
> and on the second call it adds the embargo date again to
> @embargo_release_date.  So I added a conditional that only sets
> @embargo_release_date once (to nil) if the user is trying to take
> something out of embargo.
>
> Fixing embargoable.rb caused some of the embargo tests to break even
> though the app was performing as it should.  So I tweaked the tests
> accordingly.
>
> – https://github.com/projecthydra-labs/curate/commit/9dfd0b857155f93fc4b1839dbfe8cfef48aee934

Glen, took the time to write that commit message.
When I encountered a problem with embargo, I was thankful for that message.
Glen had taken the time to explain what he was doing and why.

It gave me context to solve a subtle bug.

## Code Changes

This is very important.
Keep your changes on topic.
Which means, know what you are trying to solve and solve that.

Remember, your code is a conversation between you and the interpreter.
But future you is going to need to understand your code.
So be expressive.

Try to avoid using one letter variable names. They lack meaning.

## Automated Tests

An automated test is a way for your present self to explain how you verified the work you did.
The test should be expressive and meaningful.
It follows the same principles of the Code Changes above.

Once you have written:

1. your tests
1. then your production code
1. and got everything running

Then go back and read your tests.

* Do the tests make sense?
* Are the tests expressing your intent?
* Are the tests generating meaningful documentation?

Edit them. Clarify them. Review what they say.

## Comments

As a general rule, I don't read code comments.
My brain filters them out.

More specific, I don't read code comments that say what the code is doing.
The code is the authority on what it is doing.
Anything else is subject to being incorrect.

But when I find a comment that explains why the code was written, I am always thankful.

Tell me why you chose:

* an obtuse port
* a confusing looping structure
* a particular dependency
* a dense method

## Conclusion

Take time to engage in the conversation that is your code.
If the conversation is confusing, work to disentangle the confusing parts.
