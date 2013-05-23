---
author:   jeremyf
category: practices
filename: 2013-05-23-active-record-attributes.md
layout:   post
tagline:  Understanding ActiveRecord::Base attributes
title:    ActiveRecord::Base Attributes
tags:     rails, activerecord
---

This is a follow-up to the extra credit challenge of the Railsbridge Tutorial:
Ordering topics by vote count.

This blog post is building on the [Railsbridge Curriculum](http://curriculum.railsbridge.org/curriculum/curriculum), and will assume you've stepped through that.

## Attributes

When you load an ActiveRecord model from the database (i.e. Topic.find(1)), behind the scenes ActiveRecord is assigning the values of each column of the database table to the model's attributes.

A model's attributes are a Hash – a Dictionary of terms/definitions also known as keys/values- in which you can look up a term/key (i.e. "title") and get the term/key's value (i.e. "Should we go to lunch?").

In the terminal make sure you are in your Rails project's directory.

    $ rails console
    > Topic

### Seeing the database columns for an ActiveRecord model class

The output will look like this:

    => Topic(id: integer, title: string, description: text, created_at: datetime, updated_at: datetime)

Inside the paranthesis is a Hash with database column names for keys (i.e. id:, title:) and their respective column types as values (i.e. integer, string).
By default these are the available attributes for a Topic.

### Seeing the database column values for a new ActiveRecord model instance

Still inside the `rails console` type the following and note the results:

    > Topic.new.attributes
    => {"id"=>nil, "title" => nil, "description" => nil  "created_at"=>nil, "updated_at"=>nil}

The attributes for a new Topic are a Hash with the database column names and nil values.

### Seeing the database column values for an existing ActiveRecord model instance

Assuming you have at least one topic in your topics database table, type the following into your `rails console` and note the results (your values may vary):

    > Topic.first.attributes
    => {"id"=>1, "title" => "First Topic!", "description" => "We want to talk about this"  "created_at"=>Wed, 22 May 2013 21:51:49 UTC +00:00, "updated_at"=>Wed, 22 May 2013 21:51:49 UTC +00:00}

## Going further with Attributes

The attributes for a Topic are determined based on the SQL query used to load that Topic from the database.
Which means if we write a custom query, we can add or remove columns from our attributes Hash.

Again from the `rails console` type the following:

    > Topic.find_by_sql("SELECT id FROM topics").first.attributes
    => {"id" => 1}

This attributes Hash only contains one key/value pair: `"id" => 1`. The title, description, created_at, and updated_at attribute keys are gone.

Now, lets add a new attribute that isn't a column in the topics database table (assuming your first topic has 2 votes).

    > Topic.find_by_sql("SELECT topics.id,(SELECT count(votes.topic_id) FROM votes WHERE votes.topic_id = topics.id) AS votes_count FROM topics").first.attributes
    => {"id"=>1, "votes_count"=>2}

The above query is selecting the `id` column of the topics database and is counting the number votes of votes for each topic and assinging that result to the `votes_count` query column.

## RailsBridge Adjustments
<script src="https://gist.github.com/jeremyf/5636323.js"></script>
