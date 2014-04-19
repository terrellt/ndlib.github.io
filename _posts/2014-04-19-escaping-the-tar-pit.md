---
author:   jeremyf
category: practices
filename: 2014-04-19-escaping-the-tar-pit.md
layout:   post
tagline:  Moving Forward on Product Development
title:    Escaping the Tar Pit
tags:     ruby, rails
---

What follows is an unusually long blog post.
It is a mix of observations and calls to action.

## Consider the Introductions

As the technical lead for the Hydramata project, I've been at the intsection of two particular problems:

1. New developers joining the project don't have a strong sense of where to begin.
2. Each institution has nuanced requirements that are not necessary for other institutions.

I'd like you to take a moment and think about how you'd introduce a new developer to an application you are working on?

Where is the first place you would send them?

Did you say the:

* documentation?
* acceptance tests?
* unit tests?
* models?
* database diagram?
* routes?
* controllers?
* demo application?
* wiki?

What if you were introducing a new non-technical person? Where would you send them then?

What if you were attempting to recruit a new participant institution? Where would you send them?

How about if you were hiring a freelancer to help get you to a release?

## The Folly of Documentation

It is my strong opinion that the further documentation is away from the execution of that which it is documenting, then the less accurate the documentation is.

In other words, if the documentation is not auto-generated nor auto-verified, then it will lie.
Because the underlying code changes.
And there is no relation of the code to the documentation.

Software is soft.
And malleable.
Our documentation strategy must account for that.

My understanding of the goal of documentation is to solve the problem of "how something is done."
Code says "how something is done;" But it may not be doing a good job conveying that.

### Tests are Documentation

Tests are another form of documentation.
They may also not be doing a good job conveying that information.
But tests are declaring how the system functions.
So at least they are less likely to lie to you!

There may be a few reasons for this. Consider the following statement that I've heard several times:

> Its working, now I need to write a test.

How does this person know its working?
They likely implemented the solution, manually tested it along the way.

The resulting test that was written will be informed by the implementation.
That is to say in writing the test you may take shortcuts or make logical leaps because you have already "solved" the problem.

Another reason is that there are too many collaborators required for the test.
I have to create a two Users, a Group, a Collection, and Work to see if the first User can add a Work to a Collection that is owned by a Group who's members only include the second User.

Another is the structure of tests.
Which tests are there for a requested feature?
And which parts of those tests are "required?" (i.e. is a CSS selector required? How about a specific label name?)

## Consider the Responsibilities

Consider a "traditional" Rails controller.
It is responsible for a lot of things.

* All of the :before_filters?
* The number of objects created for an action?
* Parameter filtering and knowledge of what parameters are acceptable?
* Caching?
* ETags?
* Collaborating classes?
* HTTP responses (i.e. 302, 200, 404, etc)?
* Human responses - "Successfully created a Widget"?

A few years ago the Rails community clamoured for skinny controllers and fat models.
Extract the complexity to the models.
Then came the movement to put models on a diet as well.

The tragedy of Model/View/Controller, especially in the Rails implementation, is the implaction that there are only three types of objects.

In reality we should be working towards more types. Examples include, but are not limited to: Presenters, Service objects, Query objects, Workers, etc.

## An Example in the Wild

The late Jim Weirich had begun work on disentagling a Rails application.
This was the natural extension of his teaching and interactions with numerous developers.
Take a look at his [unfinished Wyriki project](https://github.com/jimweirich/wyriki).

Want to know where to get started? Take a look at his [WikiRepository object](https://github.com/jimweirich/wyriki/blob/master/app/services/wiki_repository.rb). It is in the `app/services/` directory of a Rails application.
The `WikiRepository` object is composed of four modules:

```ruby
class WikiRepository
  include Repo::UserMethods
  include Repo::WikiMethods
  include Repo::PageMethods
  include Repo::PermissionMethods
end
```

A quick look at the [Repo::PageMethods module](https://github.com/jimweirich/wyriki/blob/master/app/services/repo/wiki_methods.rb) expresses the intent of Pages.

```ruby
module Repo
  module PageMethods
    def find_wiki_page(wiki_id, page_id)
      wiki = Wiki.find(wiki_id)
      page = wiki.pages.find(page_id)
      Biz::Page.wrap(page)
    end

    def find_page_on(wiki, page_id)
      Biz::Page.wrap(wiki.pages.find(page_id))
    end

    def find_named_page(wiki_name, page_name)
      Biz::Page.wrap(Page.by_name(wiki_name, page_name))
    end

    def new_page_on(wiki, attrs={})
      Biz::Page.wrap(wiki.pages.new(attrs))
    end

    def save_page(page)
      page.data.save
    end

    def destroy_page_on(wiki, page_id)
      page = wiki.pages.find(page_id)
      page.data.destroy
      nil
    end
  end

end
```

How did I know to go there? I didn't.

For some reason, I started digging around in the controllers.
I found my way to the `app/services` directory.

I suppose I could've looked in the `features` directory to review the acceptance tests.
This would've given me a different perspective; one that documented how the system worked.

## Proposals

### A Reading List

I read in one sitting - the fligth from South Bend to Minneapolis - [Corey Haines's "Understanding the 4 Rules of Simple Design"](http://leanpub.com/4rulesofsimpledesign). He dives into the well established four rules, as earlier codified by Kent Beck:

1. Tests Pass
2. Express Intent
3. No Duplication (of knowledge)
4. Small

This book should be required reading.
He provides examples and subtle insights into the 4 Rules.

Please consider buying it.

### Code Retreat

A sponsored Hydra code retreat.
This could be done via remote pairing - though I'm a big proponent of face to face developer time.
Or as part of Hydra Connect.

Could we get someone with previous experience facilitating a Code Retreat?

### Acceptance Tests

> "Acceptance tests [are] tests written by a collaboration of the stakeholders and the programmers in order to define when a requirement is done...
> The purpose of acceptance tests is communication, clarity, and precision.
> By agreeing to them, the developers, stakeholders, and testers all understand what the plan for the system behavior is."
> -- Martin, Robert C. (2011-05-13). "The Clean Coder"

Acceptance tests are extremely valuable.
We should take the time to craft these.

> "Acceptance tests should always be automated.
> There is a place for manual testing elsewhere in the software lifecycle, but these kinds of tests should never be manual.
> The reason is simple: cost."
> -- Martin, Robert C. (2011-05-13). "The Clean Coder"

As a developer I believe these should be automated.
After all, if I'm working on something and changed things, I should know if what I did was not acceptable.

It is expensive to manually verify the acceptance tests.
Our Product Owners don't have time for that.
Our developers need to know quickly if a change that was introduced was unacceptable.

To help introduce a new member to the project, have them read the acceptance tests.

#### What to Test for Acceptance

I don't know if the acceptance tests should be run against the web application?
Or if they should be run against the service layer?

That is to say should our acceptance tests include language like "click", "visit", and "login".
There are two camps, and those driving the web application invariably have an additional hoop or two to jump through.

I also believe the acceptance tests should be clearly delineated.

My original intent in the Curate gem was to have a single "End to End" feature test that clicked through much of the system.
I failed to convey that intent.
Now we have feature tests that may or may not be testing something for acceptance.

#### Automating the On Boarding

One interesting possibility of writing acceptance tests with the "click", "visit", etc. terminology is that we could help with the onboarding process.

First, we set the tests to render in a "normal" browser.
As these tests run, you can see the computer clicking and filling in fields in the browser.

Second, we provide a means of explaining the spec that is being run and allowing it to be re-run.
This might mean before each test runs we display to the user the acceptance test text, then ask for input; (i.e. Back one, Run this one, Skip this one).

Third, we provide a means of slowing down the visual steps.
Things can happen really quickly, and a user should be able to run a spec slower to capture what is happening.

I'm not entirely certain if this would be valuable, but many of the requisite parts for this functionality are present.

### Many more Types of Objects

Each layer in our application stack should have a clear responsibility.
The controllers and models are really overworked.

Lets look for guidance from the Rails community, and elsewhere, on how to separate our layers.

I really like Jim Weirich's approach in his Controller / Runner interaction:

* A controller handles the parameters, invoking a runner
* A runner collaborates various services then informs the controller via a callback
* A controller handles the appropriate callback

From [Jim Weirich's Wyriki PagesController](https://github.com/jimweirich/wyriki/blob/master/app/controllers/pages_controller.rb#L1-L19)
```ruby
class PagesController < ApplicationController
  include PageRunners

  def show
    @wiki, @page = run(Show, params[:wiki_id], params[:id])
  end

  def show_named
    run(ShowNamed, named_params[:wiki], named_params[:page]) do |on|
      on.success { |wiki, page|
        @wiki = wiki
        @page = page
        render :show
      }
      on.page_not_found { |wiki_name|
        redirect_to new_named_page_path(wiki_name, named_params[:page])
      }
    end
  end
  # ...
end
```

There are no `:before_filter` declarations.
There are two objects created, though [Sandi Metz advocates that only one object be assigned](http://robots.thoughtbot.com/post/50655960596/sandi-metz-rules-for-developers) as an instance variable for a controller.
But we haven't explicitly stated what those objects are (i.e. `@wiki = Wiki.find(params[:wiki])`)

We have expressed, via the callbacks, what our expectations are.
I find reading this to be somewhat conversational.

And if another institution wants to change the `show_named` action, they could replace the PageRunners module with their own.
The new runner could do pretty much anything.

I've explored the above method in an existig project.
What I found was that my controller tests became so much easier to test.
And the tests were fast.

## Further Reading

* Martin, Robert C. "The Clean Coder: A Code of Conduct for Professional Programmers (Robert C. Martin Series)"
* Corey Haines. "Understanding the Four Rules of Simple Design"
* Sandi Metz "Practical Object-Oriented Design in Ruby"