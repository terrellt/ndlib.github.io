---
author:   jeremyf
category: practices
filename: 2014-11-05-rails-come-for-the-activerecord-stay-for-the-testing.md
layout:   post
tagline:  Coherent Development (Part 5)
title:    "Rails: Come for the ActiveRecord, Stay for the Testing"
tags:     rails, design, projecthydra, coherent-development
---

*Part of an ongoing series for [Coherent Development](/practices/coherent-development-part-1/)*

I adopted Ruby on Rails in late 2005.
It was one week after the [Snakes and Rubies seminar](https://www.youtube.com/watch?v=cb9KDt9aXc8) at DePaul University in Chicago.

Snakes and Rubies was a presentation by Adrian Holovaty and David Heinemeier Hansson.

Adrian made a fantastic pitch for Django.
Quickly make dynamic web sites.

David made some semantic jabs at Adrian regarding Web Site versus Web Application.
David's pitch was spot on.
Quickly make dynamic web applications.

As I've reflected on those two presentation over the years, I often wonder what would've happened if my company had chosen Django? Why did we choose Rails over Django?

At the time it came down to Rails being post v1.0 and Django being v0.9x.

But I suspect that was a rationalization.
The seminar was attended by lots of developers.

I believe that David's presentation was directed at developers. Adrian's spoke at a different level, as though speaking to a business manager.

And in the famous sweaty words of Steve Ballmer:

> [Developers! Developers! Developers!](https://www.youtube.com/watch?v=8To-6VIJZRE)

## Adopting Rails

From a quick survey, scaffolding could've been seen as the killer feature of Rails.
It was a tool to help make quick models, views, and controllers.

But from Adrian's presentation, I would've argued that Django was better and quicker at getting you further than Rails scaffolds.

What spoke to me was ActiveRecord.
It was a powerful abstraction.

We were on a greenfield web application project that had 100 or so database tables.
ActiveRecord was going to save us so much time.

We pounded out our models then moved up into our controllers.
There were rough patches, but ActiveRecord was helping us model our domain.

I wasn't writing tests. I felt as though I couldn't learn both Rails and the paradigm of testing.
And that was a huge mistake.

As Rails updated even patch level versions, things broke.
As our client changed scope, things broke.
As we further refined our understanding, things broke.

## Adopting Testing

We switched and began writing automated tests.
It was bumpy but my understanding of software development improved.

I realized that ActiveRecord callbacks were both powerful and painful.
When an object was saved, all of the requisite things happened.
But I had a hard time verifying the order of events.
And the conditional logic on save was terrible.

In writing tests, I was starting to eat my own dog food.
I was writing a consumer of the interface of my objects.
And I was realizing a problem: ActiveRecord callbacks get in the way of isolated testing.

But I kept using ActiveRecord callbacks.
They are stupid easy to write in the moment.
But they are painful to later maintain.

However, my tests were adding value.
They were:

* Catching regressions
* A place to write a test case to verify a bug had been squashed
* Helping me think about how I'm using the production code

*Note: I still wasn't doing test driven development. That came later.*

## My Kingdom for a Place to Call Home

In hindsight, the reason for fat models was that I didn't feel like I had an acceptable place to do the other work.
And [Jamis Buck's Fat Models, Skinny Controllers blog post](http://weblog.jamisbuck.org/2006/10/18/skinny-controller-fat-model) help crystalize this mindset.

Rails provided four application level concepts (though we now have `app/assets`):

* Views
* Controllers
* Helpers
* Models

If the code didn't go in the Models, then where else should it go?

Mercifully, I had identified early on that Controllers already did too much work.
But this kind of code didn't belong in the Views nor Helpers.

So it was left in the models.
And those models grew as they were responsible for:

* Conditional Validations
* Conditional Callbacks
* State Machines
* Relationship Managing Functions
* Queries

So while my tests `test/unit` were called Unit tests, they were a far cry from isolated.
They were entwined, lumbering, labyrinthine monsters of Doctor Frankenstein.

It was through testing and the insight of others that I realized something:

**Rails is great for making your own personal application.**
**But it implies that everything belongs in one of the top-level buckets.**
**Which means collaborative development is a potential nightmare, as objects end up with many hands touching them.**

## Adopting a Hostile Stance towards Rails

It has become clear that I need a place for each of those things.
And that is not in the model. Nor in the even worse `app/model/concerns`.

During development it may appear easier to keep these concepts together.
But the ongoing maintenance costs and enhancement costs tell a different story.
I have been down that path.
I hate it.

And all of these lessons were because I kept testing the code I was writing in Rails.
I started writing my tests first; then my production code.

Through repeated testing, my understanding of how to better construct things has evolved.

I have been tasked with helping construct sharable and configurable components for multiple applications spanning multiple development shops.

Its daunting in Rails because the "Rails Way" is easy, comfortable, and much of the ecosystem guides me to those fat and overworked models.

So thank you Rails for your implementation of the ActiveRecord pattern.
It's a fantastic pattern.
But I'm ready to have those models do less work.

And even more than that, thank you Rails for championing testing.
Without testing, I would be doomed to repeat my mistakes.

I have learned invaluable lessons from testing:

* How to verify that something I did worked
* How to pick something apart
* How to play towards a solution
* How to recognize usage patterns

And in a twist of fate, my tests tell me that Rails as written is inadequate for the solutions I need to deliver.
