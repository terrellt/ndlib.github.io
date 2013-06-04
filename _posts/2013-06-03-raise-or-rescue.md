---
author:   jeremyf
category: practices
filename: 2013-06-03-raise-or-rescue.md
layout:   post
tagline:  When and Where to Rescue the Exception
title:    Raise or Rescue
tags:     ruby, rails
---

In my [previous post](./2013-06-03-object-interfaces.md), I created the Book and Citation class, repeated here.

    Book = Struct.new(:title, :author, :publisher)

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

    > confident_ruby = Book.new("Confident Ruby", "Avdi Grimm", "Shiprise Media")
    #<struct Book title="Confident Ruby", author="Avdi Grimm", publisher="Shiprise Media">

From the above code, we can create a Citation for a Book.
What if we were to create a Citation for a Dog?

    > Dog = Struct.new(:name)
    > lassie = Dog.new('Lassie')
    > Citation.new(lassie)
    NoMethodError: undefined method `title' for #<struct Dog name="Lassie">

Excellent, a `NoMethodError` exception is raised. In other words we can't cite a dog.

But what if we attempt to cite a book in which the author is not given?
Should we raise an exception?
Probably not.

What if the citation format is extremely strict, that is an author is required for the citation.
Then perhaps.
But we must be careful, as every exception that we raise must be dealt with.

## Handling Exceptions

Broadly speaking there are two types of exceptions.
One in which the end user of your system can do something about, and one in which they are helpless.
For cases in which they are helpless, the exception is likely a bad bit of logic; Unexpected objects being passed, methods missing, etc.
In cases where they aren't helpless, they may need to make adjustments to the data.

Let's adjust our Citation to raise an exception if the title is empty (e.g. `nil`, `""`, or `[]`).
I'm also introducing a Rails controller to highlight how the end user will interact with our application.

    class Citation
      class InvalidCitationData < RuntimeError
      end
      attr_reader :citable, :authors, :publisher
      def initialize(citable)
        @citable = citable
        @title = String(citable.title)
        @authors = Array(citable.author)
        @publisher = String(citable.publisher)
      end

      def to_s
        "\"#{title}\" by #{authors.to_sentence}. #{publisher}."
      end

      def title
        if @title.empty?
          raise InvalidCitationData, "Expected a :title for Citation"
        end
        @title
      end

    end

    class CitationsController < ApplicationController
      rescue_from Citation::InvalidCitationData do |exception|
        redirect_to edit_book_path(book), notice: "Could not show citation. #{exception.message}"
      end

      rescue_from NoMethodError do |exception|
        send_error_notification_to_team!
        render text: "We're sorry something went wrong. We are working on it.", status: 500
      end

      def show
        @citation = Citation.new(book)
        render 'show'
      end
      protected
      def book
        @book ||= Book.new("", "Avdi Grimm", "Shiprise Media")
      end
    end

in ./app/views/citations/show.html.erb

    <p>Citation: <%= @citation %><p>

The `<%= @citation %>` will automatically coerce `@citation` into a string by calling the `#to_s` method.

We now have two different exceptions that can be raised.

* `NoMethodError` if the citable object doesn't have a `#title`, `#author`, or `#publisher` method.
* `Citation::InvalidCitationData` if the citable object's title is empty? (e.g. `nil`, `''`, or `[]`).

By differentiating the types of exceptions, we can handle each case separately.
In the case of a Book that doesn't have a title, we are rescuing that exception with the `rescue_from Citation::InvalidCitationData` directive.
The end user can perhaps fix the book.

If for some reason the Book didn't implement the `#title` method and we attempted to generate the citation, we would get a `NoMethodError`.
In fact, no amount of end user data manipulation would be able to correct this.
So we `send_error_notification_to_team!` and render some text apologizing.
After all, someone on the development team may have improperly implemented the Book interface.

# Further Reading

["Exceptional Ruby" by Avdi Grimm](http://)
[Rails's :rescue_from method](http://guides.rubyonrails.org/)