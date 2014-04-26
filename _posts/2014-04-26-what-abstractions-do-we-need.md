---
author:   jeremyf
category: practices
filename: 2014-04-26-what-abstractions-do-we-need.md
layout:   post
tagline:  ...and are we using the right ones?
title:    What Abstractions Do We Need?
tags:     ruby, rails
---

Project Hydra has attempted to cleave close to Rails patterns.
In doing so, the hope is that new adopters *need only learn Rails* to get started in Hydra.

But does Rails offerings hold true to our needs?

## Evaluating the stack

### Model

Below are the two requisite components:

* Active Model - ["provides a known set of interfaces for usage in model classes."](https://github.com/rails/rails/blob/adding-rails-template-working-directory-other/activemodel/README.rdoc#L3)
* Active Record - ["connects classes to relational database tables to establish an almost zero-configuration persistence layer for applications."](https://github.com/rails/rails/blob/adding-rails-template-working-directory-other/activerecord/README.rdoc#L3-L4)

#### Active Model

Active Model provides a plethora of concerns (eg. validation, attribute definition, callbacks, model naming, serialization, etc.) that you can mixin to your models.
Active Record makes heavy use of those mixins.

Active Model also provides the somewhat obscure [ActiveModel::Lint module](http://github.com/rails/rails/blob/HEAD-other/activemodel/lib/active_model/lint.rb).

> You can test whether an object is compliant with the Active Model API by including `ActiveModel::Lint::Tests` in your TestCase.

#### Active Record

When I was reviewing whether to adopt Rails, Active Record was the killer pattern.
You can Active Record to describe your database relationships.

Active Record supports numerous adopters to different databases (eg Oracle, SQLite, MySQL, Postgresql, etc.). AREL provides an abstraction layer between Active Record and the numerous databases.

Arel is a SQL AST (abstract syntax tree) manager for Ruby.
It is my conjecture, though it may be crazy talk, that an Arel adapter could be created for Solr and Fedora.
That adapter may not be "complete" by other standards, but it would mean we could lean on ActiveRecord for our data modeling instead of standing up a separate Active Fedora project.

So why not create the adapter?
It appears non-trivial to implement a non-SQL adapter for Arel.
And I don't believe the ActiveRecord pattern is applicable for what we do.
More on that later.

### View

* Action View - ["a framework for handling view template lookup and rendering, and provides view helpers that assist when building HTML forms, Atom feeds and more."](https://github.com/rails/rails/blob/master-other/actionview/README.rdoc#L3-L4)

ActionView's template lookup mechanism is important in sharing views.

### Controller

* Action Pack - [""]

### Bonus

* Active Support - [""]
* Action Mailer - [""]

