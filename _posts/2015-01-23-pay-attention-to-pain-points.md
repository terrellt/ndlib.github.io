---
author:   jeremyf
category: practices
filename: 2015-01-23-pay-attention-to-pain-points.md
layout:   post
tagline:  Coherent Development (Part 7): View Testing
title:    Pay Attention to Pain Points
tags:     rails, design, projecthydra, testing, coherent-development
---

*Part of an ongoing series for [Coherent Development](/practices/coherent-development-part-1/)*

## Some Background

In a recent conversation with [Dan](http://dan.brubakerhorst.com/), my coworker and supervisor, I asked for his impression of me as a team member and developer. His response was something along the lines of:

> Jeremy, I have told others that you are our canary in a coalmine.
> That is to say that you are aware of problems, and experience the pain of those problems, well before many other people.

A fair summary. I often don the chicken suit and proclaim the sky is failing.
I try to temper that, but its hard. I recognize and intuit patterns as abstractions come screaming at me.

Later we had a discussion about ongoing work for one of our applications.
He had just finished major rework of the views.

He said he would've spent about an afternoon to make the behavior and visual adjustments.
But he ended up spending *three or four* days fighting with our UI integration tests.

There are few reasons why this protracted skirmished happened:

1. There are too many UI driven integration tests.
2. The UI tests drive on CSS selectors, often times nested

Those were some of his pain points.

Our conversation shifted as we began discussing how to get ahead of that problem for [Sipity, our upcoming application](https://github.com/ndlib/sipity).

## Problem 1: There are Too Many UI Driven Integration Tests

I'm going to keep this one short. If you have lots of UI driven integration tests, you are coupling your application's API to current state of your web application's presentation.

In other words, consider making an Action object that your controller's action method uses.
By teasing apart that one concept, you now have a robust collection actions that you can bombard without consideration for the UI.

This is a painful lesson we have learned and it is informing our current development.

## The UI Tests Drive on CSS Selectors

This problem cannot be solved by teasing apart action objects.

It requires thinking about the HTML document and being mindful of [Single Responsibility Principle](http://en.wikipedia.org/wiki/Single_responsibility_principle).

Consider the consumers/users of an HTML document:

* CSS
* Javascript
* Humans (with different languages)
* Search Engines
* Automated Testing
* Screen readers and assistive devices

In other words, there are lots of reasons for an HTML page to change.

* New behavior
* Search engine optimization
* New structure
* Removing content
* Semantic improvements
* Improving accessibility
* and more

The goal is to minimize the use of things that will change.

## A Page Object Diversion

I had heard about PageObjects, perhaps first from [ThoughtBot's blog post](http://robots.thoughtbot.com/better-acceptance-tests-with-page-objects).

I use objects to give predictability, shape, and context to concepts.
So PageObjects made sense.
With an object I can isolate the impact of change.

I recommend that anyone using Capybara for UI testing make use of the [SitePrism](https://github.com/natritmeyer/site_prism) gem.

> A Page Object Model DSL for Capybara
>
> SitePrism gives you a simple, clean and semantic DSL for describing your site
> using the Page Object Model pattern, for use with Capybara in automated
> acceptance testing.
>
> Find the pretty documentation here: http://rdoc.info/gems/site_prism/frames
>
> From [https://github.com/natritmeyer/site_prism](https://github.com/natritmeyer/site_prism)

Please take some time to read up on both PageObjects and SitePrism.

## Back to the Testing

Thank you for taking some time to read up on the PageObjects and SitePrism.

As Dan and I were talking about UI testing and CSS, there was a sticking point.

I needed to click on a link for a TODO item; The link had a generic text and there could be more than one.

Here is a snippet from a Work page's HTML. I wrote it quickly as I want to demonstrate to Dan how the TODO list was generated. This code represents shifting a hard-coded action list to a dynamic action list.

```rails
<ul>
  <%- model.actions.each do |action| -%>
    <li class="required-<%= action.name %>">
      <span><%= action.status %></span>
      <span><%= action.label %></span>
      <%= link_to 'Do it!', action.path %>
    </li>
  <%- end -%>
<ul>
```

I had a test, with a PageObject that already tested clicking on the hard-coded version.

Below is the snippet of a [SitePrism](https://github.com/natritmeyer/site_prism) PageObject that I used.

```ruby
class WorkPage < SitePrism::Page
  def click_required(name)
    find(".required-#{name.downcase} a").click
  end
end
```

Notice the `.required-*` selector. This selector was ripe for change.

We wanted to move our automated UI integration tests away from CSS class selectors; After all our CSS elements would change as we modified the page for new behavior and a modified structure.

We kicked around a few options:

* namespaced CSS classes (i.e. test-*)
* data attributes (i.e. test-data-*)

Intuitively both Dan and I felt those options were awkward, arbitrary, and not close to any standard.

As our conversation continued, Dan made an off-handed comment about schema.org.
And something clicked.

## Make Objects on Pages

Why don't we make a [schema.org/Action](http://schema.org/Action) to represent that concept.
The general conjecture is to make objects to insulate against change.
We also give greater meaning to those elements without getting entangled in the volatility of behavior and styling.

_You can argue if this is the correct meaning, but I'll settle for isolating that decision within an object._

First review the [SitePrism based feature test](https://github.com/ndlib/sipity/blob/eb0f8c958aa2d8e2f2107ffe60f437f15890a47f/spec/features/sipity/create_minimum_viable_work_spec.rb#L51-L75).

```ruby
feature 'Minimum viable SIP', :devise do
  scenario 'User can describe additional data' do
    login_as(user, scope: :user)
    visit '/start'
    on('new_work_page') do |the_page|
      expect(the_page).to be_all_there
      the_page.fill_in(:title, with: 'Hello World')
      the_page.select('etd', from: :work_type)
      the_page.choose(:work_publication_strategy, with: 'do_not_know')
      the_page.submit_button.click
    end

    on('work_page') do |the_page|
      the_page.click_todo_item('todo>required>describe')
    end

    on('describe_page') do |the_page|
      expect(the_page).to be_all_there
      the_page.fill_in(:abstract, with: 'Lorem ipsum')
      the_page.submit_button.click
    end

    on('work_page') do |the_page|
      expect(the_page.todo_item_named_status_for('todo>required>describe')).to eq('done')
    end
  end
end
```

My intention is that this test is as clear as possible in its intent.
That someone who is used to interacting with a web page might be able to read this test and understand what is happening.

Now review the [snippet extracted from the Sipity view code](https://github.com/ndlib/sipity/blob/eb0f8c958aa2d8e2f2107ffe60f437f15890a47f/app/views/sipity/controllers/works/show.html.erb#L47-L52)

```rails
<ul>
  <%- actions.each do |action| -%>
    <li itemscope itemtype="http://schema.org/Action">
      <meta itemprop="name" content="todo><%= set %>><%= action.name %>">
      <span itemprop="actionStatus"><%= action.state %></span>
      <span itemprop="description"><%= action.label %></span>
      <a itemprop="url" href="<%= action.path %>">Do it!</a>
    </li>
  <%- end -%>
</ul>
```

This is the markup that I chose to reflect the action object.
To reiterate, it:

* does not collide with traditional CSS selectors
* is meaningful to assistive devices

And finally, below is the [snippet extracted from the Sipity page object](https://github.com/ndlib/sipity/blob/eb0f8c958aa2d8e2f2107ffe60f437f15890a47f/spec/support/site_prism_support.rb#L71-L85).

```ruby
class WorkPage < SitePrism::Page

  def click_todo_item(name)
    find_named_object(name).find("[itemprop='url']").click
  end

  def todo_item_named_status_for(name)
    find_named_object(name).find("[itemprop='actionStatus']").text
  end

  private

  def find_named_object(name)
    object_name_node = find("[itemprop='name'][content='#{name.downcase}']")
    # Because Capybara does not support an ancestors find method, I need to
    # dive into the native object (i.e. a Nokogiri node). The end goal is to
    # find the named object element and thus be able to retrieve any of the
    # underlying attributes.
    parent_ng_node = object_name_node.native.ancestors('[itemscope]').first
    find(parent_ng_node.path)
  end
end
```

I have attempted to give rich meaning to Action object, by explaining via method names what is happening.
It took a few revisions but I finally was able to get a general case for handling schema.org within Capybara.
