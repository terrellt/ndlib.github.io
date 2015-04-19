---
author:   jeremyf
category: practices
filename: 2015-04-19-leveraging-instance-variables-in-ruby-classes.md
layout:   post
tagline:  Hint - Prefer Methods over Instance Variables
title:    Leveraging Instance Variables in Ruby Classes
tags:     ruby, rails, design, coherent-development, novice
---

For the past 8 months, I've been mentoring for [The Firehose Project](http://www.thefirehoseproject.com/); an online Rails bootcamp.

What follows is a rudimentary introduction to leveraging instance variables in Ruby.
Along the way, I'll talk about my preferred class methodology as it relates to setting instance variable vs. calling setter methods.

```ruby
# Example 1
class Book
  def initialize(a_title)
    @title = a_title
  end

  def title
    @title
  end
end
```

This can be condensed into the following:

```ruby
# Example 2
class Book
  def initialize(a_title)
    @title = a_title
  end

  attr_reader :title
end
```

More and more I prefer to not set instance variables as part of initialization, but instead call setter methods (more on that later).

```ruby
# Example 3
class Book
  def initialize(a_title)
    # Make sure to remember to use `self.title = a_title` instead of
    # `title = a_title`. If you use `title = a_title`, you will be setting the
    # local variable `title` instead of calling the instance method `#title=`
    # (and thus setting the instance variable `@title`)
    self.title = a_title
  end

  attr_reader :title

  def title=(a_title)
    @title = a_title
  end
end
```

This can be condensed into the following:

```ruby
# Example 4
class Book
  def initialize(a_title)
    self.title = a_title
  end

  attr_reader :title
  attr_writer :title
end
```

And further into:

```ruby
# Example 5
class Book
  def initialize(a_title)
    self.title = a_title
  end

  attr_accessor :title
end
```

Using `attr_accessor` and instance method setters on initialize is my preferred methodology (as in Example #5).
Previously, I've used Example #2 as my defacto methodology.
However, I'm less inclined to use that these days.

My reasoning for using a setter method is as follows:

* I am encapsulating how the instance variable is ultimately set.
* I am easing any work that may need to be done to extend the Book class.

Consider the following:

```ruby
# Example 6
class FrenchBook < Book
  def title=(a_title)
    @title = translate_to_french(a_title)
  end
end
```

In using the setter method, the setter method is an inflection point – a place to extend and modify - in the code.

If I were setting the instance variable in the initialize method, my inflection point would be the initialize method.
The following example illustrates that change.


```ruby
# Example 7
class Book
  def initialize(a_title)
    @title = a_title
  end
  attr_reader :title
end

class FrenchBook < Book
  def initialize(a_title)
    @title = translate_to_french(a_title)
  end
end
```

The overall lines of code for both examples are the same.
But consider what happens when we need to change the `Book`class.


```ruby
# Example 8
class Book
  def initialize(a_title, an_author)
    # do stuff here
  end
end
```

The `FrenchBook` class of Example 7 would need to change.

## Conclusion

Using `attr_writer` instead of setting an instance variable is a quick means of separating the concerns of your methods. Just as a class should have a [single responsibility](http://en.wikipedia.org/wiki/Single_responsibility_principle) (i.e. a reason to change), so too should your methods.

By using a method to set your instance variable, you are making it easier for extension, modification, and overall maintenance.

What follows is my preferred class structure these days.

## Example 9

```ruby
class Book
  def initialize(a_title)
    self.title = a_title
  end

  attr_accessor :title
  private :title=
end
```

Outside of object instantiation, I don't want to expose any means of updating the title.
I want to protect from outside forces the mutability of the object's state.