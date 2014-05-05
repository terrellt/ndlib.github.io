---
author:   jeremyf
category: practices
filename: 2014-05-05-should-we-stop-implementing-behavior-methods-in-active-record-pattern-objects.md
layout:   post
tagline:  We already ask so much of them
title:    Should we stop implementing behavior methods in Active Record pattern objects?
tags:     ruby, rails
---

Consider the following code:

```ruby
class Page
  def add_an_editor(person)
    # Do it!
  end
end
```

Now add a new object that we want to add editors to:

```ruby
class Section
  def add_an_editor(person)
    # Do it!
  end
end
```