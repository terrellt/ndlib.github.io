---
author:   jeremyf
category: practices
filename: 2015-11-07-test-driven-development.md
layout:   post
tagline:  Its All About Software Cartography
title:    Test Driven Development
tags:     rails, design, projecthydra, coherent-development
---

Software development at its core is about writing maps.
Mapping one representation to another.

Writing a user interface to map someone's idea to computer input.
Mapping that input into an object in the application.
Then mapping that object to a persistence layer.

In other words: *Input* to *Process* to *Output*.

Consider what it means to be a software developer.

As a developer, my job is to implement solutions to problems.
In an ideal world, I will contribute to exploring the problem and collaborating on describing the proposed solution.

In this case the:

* *Input* is the **problem**
* *Output* is the **proposed solution**
* *Process* is **collaboration via building use cases**

Again mapping.

At a certain point in software development, an idea must be made executable.
Take a proposed solution (e.g. the idea) as *Input*, map it to the *Output* production code (e.g. the executable).
But what is the process?

Mashing on the keyboard is an answer but it is inadequate.

## Writing a Map

When writing the map from one abstraction to another, I have found it important to think about:

* Validations - how will I verify that each input component is correct
* Coercions - how will each input component be coerced into the output component
* Expressiveness - how will I make sure what is happening is as clear as possible

And when translationg from one language to another, there is opportunity for translation errors.
Even using the same language with different receivers there is opportunity for translation errors.

Do you want translation errors in your production code? **No.**
You want it to work.

My conjecture is that the *Process* should be writing automated tests.

### Acceptance Tests

> We will define acceptance tests as tests written by a collaboration of the stakeholders and the programmers in order to define when a requirement is done.
>
> ...
>
> Acceptance tests should always be automated.
> There is a place for manual testing elsewhere in the software lifecycle, but these kinds of tests should never be manual.
> The reason is simple: cost.
>
> *The Clean Coder: A Code of Conduct for Professional Programmers* by Robert C. Martin

**Acceptance tests verify that what you are doing is right.**

This is where you **validate** that you are processing the proposed solution.
It is where you **express**, to the best of your ability, what is happening in the solution.
It is where you **coerce** the ideas of "as a user" and "with a valid form input" into objects in the system.

The goal is to agree on how to verify that your solution is done.

### Unit Tests

**Unit tests verify that you are doing things the right way.**

This is where you **validate** the internal processes of your solution.
This is where you **express** the pieces and parts that make up the solution.
This is where you **coerce** small inputs into small outputs.

Unit tests should be readable by developers and be treated as low-level API documentation.

### Functional Tests

These tests build the bridge between unit tests and acceptance tests.
They are a map.

This is where you **validate** that you have a cohesive neighborhood of objects.
This is where you **express** how that neighborhood behaves.
It is where you **coerce** input objects to a final assertion.

## Conclusion

The process of writing tests is a means for validating your assumptions.
It is a way of changing the conversation from "How will I solve this problem?" to "How I will I verify my solution?"

> The practioner of literate programming can be regarded as an essayist, whose main concern is with exposition and excellence of style.
> Such an author, with thesaurus in hand, chooses the names of variables carefully and explains what each variable means.
> He or she strives for a program that is comprehensible because its concepts have been introduced in an order that is best for human understanding, using a mixture of formal and informal methods that reinforce each other.
>
> "Literate Programming" by Donald Knuth

Pay attention to what your tests are saying.
Review both the names of the called methods and the names of your test methods.
Provide guidance for those working with your code.

And for those of you using Rspec, review your specs by running `rspec --format documentation`.
Ask others to review it.
Keep a conversation open.
