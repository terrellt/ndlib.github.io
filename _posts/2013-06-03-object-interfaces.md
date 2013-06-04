---
author:   jeremyf
category: practices
filename: 2013-06-03-object-interfaces.md
layout:   post
tagline:  Guarding the Borders
title:    Object Interfaces
tags:     ruby
---

I've never programmed in Java, but I've read quite a bit of technical books that use Java as examples.
In particular, I've been a bit jealous of the concept of Interfaces for Java.
The Java Interface is a way to fulfill two sides of a contract.
An object implements an interface and another object collaborates with the interface as implemented by the first object.

I've never programmed in Eiffel, but I've heard mention of its method contracts.
Namely, each method in Eiffel has a pre-condition and a post-condition.
When a method is called, before any of the method is executed, if the pre-condition is not met, an exception is raised.
This can be done inside the method, but Eiffel's structure does it outside the method; Decorating the method if you will.

I've only barely programmed in Python, but I've heard one of its killer features is Method Decorators.
Method Decorators are ways of wrapping additional behavior around a method.
They could be used to implement database transactions, retries, or pre/post conditions.

In the case of interfaces and pre/post conditions, these are codified instructions for how objects will collaborate.
They explain to the software developers the expected interactions.

Consider the following Citation object and its interaction with a Book:

    Book = Struct.new(:title, :author, :publisher)
    confident_ruby = Book.new("Confident Ruby", "Avdi Grimm", "Shiprise Media")
    puts Citation.new(confident_ruby).to_s
    > "\"Confident Ruby"\ by Avdi Grimm. Shiprise Media."

Here is a simple (and naive) implementation of a Citation.

    class Citation
      def initialize(book)
        @book = book
      end

      def to_s
        "\"#{@book.title}\" by #{@book.author}. #{@book.publisher}."
      end
    end

But not all books have one author:

    oo_software = Book.new("Growing Object Oriented-Software, Guided by Tests", ["Steve Freeman", "Nat Pryce"], "Addison-Wesley")
    puts Citation.new(gang_of_four).to_s
    > "\"Growing Object Oriented-Software, Guided by Tests"\ by Steve Freeman and Nat Pryce. Addison-Wesley"

We now have logic about the author(s) of a Book.

    class Citation
      def initialize(book)
        @book = book
      end

      def to_s
        "\"#{@book.title}\" by #{authors_for_citation}. #{@book.publisher}."
      end

      protected
      # I'm using ActiveSupport's #to_sentence method for an array
      def authors_for_citation
        Array(@book.author).to_sentence
      end
    end

After extracting the `#authors_for_citation` method, I now have two methods that know the implementation details of the book.
What happens when I want to cite a journal article? Or something significantly different from a book?

From Avdi Grimm's **"Guard the borders, not the hinterlands..."** section of **"Confident Ruby"**
<blockquote>
    Programming defensively in every method is redundant, wasteful of pro- grammer resources, and the resulting code induces maintenance headaches.
    The techniques here are best suited to the borders of your code.
    Like customs checkpoints at national boundaries, they serve to ensure that objects passing into code you control have the right "paperwork"—the interfaces needed to play the roles expected of them.
</blockquote>

The following Citation class uses the initialize method to extract the relevant attributes from the book.
In fact, instead of calling the input `book`, I re-named it `citable`.
By renaming the Citation class I am giving clues to a future developer – self included – that a Citation is for more than just a Book.

    class Citation

      attr_reader :citable, :title, :authors, :publisher
      def initialize(citable)
        @citable = citable
        @title = String(citable.title)
        @authors = Array(citable.author)
        @publisher = String(citable.publisher)
      end

      def to_s
        "\"#{title}\" by #{authors.to_sentence}. #{publisher}."
      end

    end

The above implementation ensures that the Citation object can collaborate with the Book object.
The `Citation#initialize` method defines the expected interface that a citable object should implement.


# Further Reading

"Confident Ruby" by Avdi Grimm