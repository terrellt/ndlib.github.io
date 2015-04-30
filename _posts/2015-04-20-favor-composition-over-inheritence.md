---
author:   jeremyf
category: practices
filename: 2015-04-20-favor-composition-over-inheritence.md
layout:   post
tagline:  And Avoid ActiveSupport::Concern as Its Encourages a Leaky Ship
title:    Favor Composition over Inheritence
tags:     ruby, rails, design, coherent-development
---

> Favor object composition over class inheritence
>
> "Design Patterns: Elements of Reusable Object-Oriented Software" by Gamma, Erich; Helm, Richard; Johnson, Ralph; Vlissides, John

This much quoted phrase has been written about by lots of people.
A Google search yields a plethora of results.

Bill Venner's asks the following question of Erich Gamma:

> "Favor object composition over class inheritance." What does that mean, and why is it a good thing to do?

Erich Gamma's response:

> Inheritance is a cool way to change behavior. But we know that it's brittle, because the subclass can easily make assumptions about the context in which a method it overrides is getting called. There's a tight coupling between the base class and the subclass, because of the implicit context in which the subclass code I plug in will be called. Composition has a nicer property. The coupling is reduced by just having some smaller things you plug into something bigger, and the bigger object just calls the smaller object back. From an API point of view defining that a method can be overridden is a stronger commitment than defining that a method can be called.
>
> ["Design Principles from Design Patterns: A Conversation with Erich Gamma, Part III" - Bill Venners & Erich Gamma](http://www.artima.com/lejava/articles/designprinciples4.html)

There are lots of blog posts about favoring object composition over class inheritence, but I want to go into how to do it in Ruby on Rails.

The example that I'm using is sanitized from a real-worlde example.
It highlights the many moving parts.

```ruby
class ResearchPaper
  def initialize(title:, abstract:)
    self.title = title
    self.abstract = abstract
  end

  attr_accessor :title, :abstract

  def abstract_to_html
    ConvertToHtml.call(abstract)
  end
end
```

Along comes another class with an abstract.

```ruby
class ConferencePresentation
  def initialize(title:, abstract:, presenter:)
    self.title = title
    self.abstract = abstract
    self.presenter = presenter
  end

  attr_accessor :title, :abstract, :presenter

  def abstract_to_html
    ConvertToHtml.call(abstract)
  end
end
```

There is a duplication of knowledge that is occurring.
So lets refactor.

Options available to us:

* Extract a CreativeWork abstract class (or [Wortem](https://github.com/projecthydra-labs/hydra-works/issues/8))
* Extract a module to Mixin the appropriate behavior
* Extract a collaborating class concerning abstracts then compose it back in

## CreativeWork as an Abstract Class

```ruby
class CreativeWork
  def initialize(title:, abstract:)
    self.title = title
    self.abstract = abstract
  end

  attr_accessor :title, :abstract

  def abstract_to_html
    ConvertToHtml.call(abstract)
  end
end

class ResearchPaper < CreativeWork
end

class ConferencePresentation < CreativeWork
  def initialize(presenter:, **keywords)
    super(**keywords)
    self.presenter = presenter
  end
  attr_accessor :presenter
end
```

This is great, but does it make sense for `ConferencePresentation` and `ResearchPaper` to have a common direct ancestor? To rephrase that, is there a concept between `ConferencePresentation` and `CreativeWork`? `Presentation` perhaps? Do we want to be in the business of managing huge inheritance trees?

I don't. And you shouldn't either.

## Mixin Module

Up next, the ubiquitous Module mixin, made super easy by ActiveSupport::Concern.

```ruby
module WithAbstract
  def abstract_to_html
    ConvertToHtml.call(abstract)
  end
end

class ResearchPaper
  include WithAbstract

  def initialize(title:, abstract:)
    self.title = title
    self.abstract = abstract
  end

  attr_accessor :title, :abstract
end

class ConferencePresentation
  include WithAbstract

  def initialize(title:, abstract:, presenter:)
    self.title = title
    self.abstract = abstract
    self.presenter = presenter
  end

  attr_accessor :title, :abstract, :presenter
end
```

Better, we no longer have an inheritance tree for our ConferencePresentation and ResearchPaper.
However we now have a subtle coupling; What does `WithAbstract` depend upon?
We can push the example even further leveraging `ActiveSupport::Concern`.

```ruby
module WithAbstract
  extend ActiveSupport::Concern
  included do
    def initialize(**keywords, :abstract)
      super(**keywords)
      self.abstract = abstract
    end
    attr_accessor :abstract
  end
  def abstract_to_html
    ConvertToHtml.call(abstract)
  end
end

class ResearchPaper
  include WithAbstract

  def initialize(title:)
    self.title = title
  end

  attr_accessor :title
end

class ConferencePresentation
  include WithAbstract

  def initialize(title:, presenter:)
    self.title = title
    self.presenter = presenter
  end

  attr_accessor :title, :presenter
end
```

While we have encapsulated the dependencies of the `WithAbstract`, there is a confusing interaction with the initialize method.

## Composition Module

```ruby
```

**Incomplete**
-------------------------------------------------------------


class ResearchPaper

```ruby
class RequestChangeOnBehalfOf
  include ActiveModel::Validations

  def initialize(comment:, on_behalf_of_collaborator_id:, submitted_by:, repository: default_repository)
    self.comment = comment
    self.on_behalf_of_collaborator_id = on_behalf_of_collaborator_id
    self.submitted_by = submitted_by
    self.repository = repository
  end

  validates :on_behalf_of_collaborator_id, presence: true, inclusion: { in: :valid_on_behalf_of_collaborator_ids }

  def on_behalf_of_collaborator
    repository.get_collaborator_for(id: on_behalf_of_collaborator_id)
  end

  def available_on_behalf_of_collaborators
    repository.available_on_behalf_of_collaborators(user: submitted_by)
  end

  def submit
    repository.write_comment_for(form: self)
  end

  private

  attr_accessor :comment, :on_behalf_of_collaborator_id, :submitted_by, :repository

  def valid_on_behalf_of_collaborator_ids
    repository.get_all_valid_collaborators_ids_for(user: submitted_by)
  end

  def default_repository
    CommandRepository.new
  end
end
```

[This is the commit in which I chose to craft a small collaborating object](https://github.com/ndlib/sipity/commit/3cce37579df96b4ae3f783605f1ba1d191013cc7#diff-3d600fac39c68ac9be0e872b98c2cb82) instead of using the ubiqutious ActiveSupport::Concern kludge.