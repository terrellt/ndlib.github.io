---
author:   jeremyf
category: practices
filename: 2015-10-06-active-record-pattern-and-alternatives.md
layout:   post
tagline:  Coherent Development (Part 4)
title:    ActiveRecord Pattern and Alternatives
tags:     rails, design, projecthydra, coherent-development
---

Two patterns for [Object Relational Mapping](http://en.wikipedia.org/wiki/Object-relational_mapping) that I'm familiar with are:

* ActiveRecord
* DataMapper

Rails implements the ActiveRecord pattern via the `activerecord` gem.
See [Martin Fowler's summary of the ActiveRecord pattern](http://www.martinfowler.com/eaaCatalog/activeRecord.html).

![Data & Behavior Object and Persistence Layer](/images/active-record-pattern.png)

A sibling to ActiveRecord is the DataMapper pattern:

![Data Behavior Object, Mapping Object, and Persistence Layer](/images/data-mapper-pattern.png)
See [Martin Fowler's summary of the DataMapper pattern](http://martinfowler.com/eaaCatalog/dataMapper.html)

There are a few manifestions:

* [DataMapper](http://datamapper.org/)
* [Ruby Object Mapper](http://rom-rb.org/)
* [Lotus::Model](https://github.com/lotus/model)

If you have spent much time in the Rails ecosystem, the DataMapper pattern may look a little weird.

## Breaking Down the ActiveRecord Pattern

> Rails applications are maturing, and a lot of developers are starting to realize that these patterns may not scale so well to more and more complex applications.
>
> When figuring out how to move forward, we could do much worse than to consult the same book that laid the blueprints for Rails’ architecture, [Patterns of Enterprise Application Architecture](http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420)
>
> -- [Avdi Grim's "Fowler on Rails"](http://devblog.avdi.org/2011/07/27/fowler-on-rails/)

Please take time to note the methods in both my diagram and Martin Fowler's diagram.

There are three kinds of methods:

* Attribute accessors
* Data mapping methods (i.e. insert, update)
* Derived attribute getters

What is missing is the business logic methods that are used to transform an object.

* How do I add a page to the document?
* How do I add a person to a group?
* Generate derivatives on save of the document
* Notify the person when their record has been updated
* Complex query methods

Perhaps it is the scope of the example, but I'm going to defer to Rober Martin:

> Active Records are special forms of DTOs [Data Transfer Objects]. They are data structures with public variables; but they typically have navigational methods like *save* and *find*. Typically these Active Records are direct translations from database tables, or other data sources.
>
> Unfortunately we often find that developers try to treat these data structures as though they were objects by putting business rule methods in them. This is awkward because it creates a hybrid data structure and an object.
>
> The solution, of course, is to treat the Active Record as a data structure and to create separate objects that contain the business rules and that hide their internal data (which are probably just instance of the Active Record).
>
> -- Martin, Robert C. "Clean Code: A Handbook of Agile Software Craftsmanship (Robert C. Martin Series)"

Also, notice that the ActiveRecord pattern maps to one persistence layer.
In the case of Hydra, we have conceded that there are two persistence layers for a given object:

* Fedora
* SOLR

Consider the following workflow:

I want to edit a Fedora repository object. I'm going to be saving my work as I go.

In the current Fedora model this would be creating multiple versions, which may be meaningless and full of chatter.

So instead I want to build up my changes over time, then publish them back into the repository.
It is likely that there will need to be a third persistence layer introduced into this solution.

## Breaking Down the DataMapper Pattern

The DataMapper pattern extracts the `to` and `from` persistence methods of the ActiveRecord pattern into a separate class.

I find this compelling because Hydra already has two persistence layers; And a reasonable use case for a third.
Adding multiple persistence layer coordination methods places extreme stress on the ActiveRecord object.
The object becomes more unwieldy as it already violated the Single Responsibility principle; Now its felonious.

I once told my colleage that I have never regretted making additional classes.
It is easier to collapse a trivial class's responsibilities into another class than it is to extract a class.

So these days, I err towards creating classes.

Its like LEGO blocks.
With smaller pieces you have greater building options.

Yes it may take a bit longer to build things up.
But the small parts are more reusable.

## What I'm Advocating

ActiveRecord is a useful pattern.
It has gotten us a very long way in our development cycles.

But it is a pattern that does not encourage flexibility.

> The Data Mapper is a layer of software that separates the in-memory objects from the database.
> Its responsibility is to transfer data between the two and also to isolate them from each other.
> With Data Mapper the in-memory objects needn't know even that there's a database present; they need no SQL interface code, and certainly no knowledge of the database schema.
>
> – [Martin Fowler's summary of the Data Mapper pattern](http://martinfowler.com/eaaCatalog/dataMapper.html)

Read that last line, "With Data Mapper the in-memory objects needn't know even that there's a database present."

That sounds like a dream to me.
It means the vast majority of our tests can target the in-memory object and not worry about persistence.
In other words, we need not always download and run Jetty.

In other words, lets model the data structures without a concern for persistence.

## Helping with Understanding

Reading ActiveRecord data structures is easy.
They are expressive and have proven very useful.

But to explore how things move from memory to persistence and vice versa becomes a lengthy exercise.
By adding the Mapper concept for persistence negotiation, I can point someone to how we transform the in-memory object into the persistence representation.

Over the next week I'm going to be exploring the DataMapper pattern as it manifests in the [Lotus::Model](https://github.com/lotus/model) project.
I have done something similar on the [Hydramata::Works](https://github.com/ndlib/hydramata-works) project, but it was a somewhat different concern.
