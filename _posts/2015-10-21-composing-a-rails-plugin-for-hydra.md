---
author:   jeremyf
category: practices
filename: 2015-10-21-composing-a-rails-plugin-for-hydra.md
layout:   post
tagline:  An Engine without the Antics
title:    Composing a Rails Plugin for Hydra
tags:     rails, design, projecthydra, coherent-development
---

There exists [a guide for creating a Rails Plugin](http://guides.rubyonrails.org/plugins.html).
It provides guidance but perhaps not as detailed as it could.

A Rails plugin is nothing more than a Ruby gem that has a dependency on Rails.
You can create a plugin by running the following:

```console
$ rails plugin new my_plugin
```

The following files are generated:

```console
├── Gemfile
├── MIT-LICENSE
├── README.rdoc
├── Rakefile
├── hello_world.gemspec
├── lib
│   ├── hello_world
│   │   └── version.rb
│   ├── hello_world.rb
│   └── tasks
│       └── hello_world_tasks.rake
└── test
    ├── dummy
    ├── hello_world_test.rb
    └── test_helper.rb
```

There are ample options for generating the plugin (see `$ rails plugin new -h`) but I prefer to create my plugins as a base gem, then incorporate rails.

To do this, I make the gem by using `$ bundle gem my_plugin`

The following files are generated:

```console
├── Gemfile
├── LICENSE.txt
├── README.md
├── Rakefile
├── hello_world.gemspec
└── lib
    ├── hello_world
    │   └── version.rb
    └── hello_world.rb
```

They are similar but a few less assumptions are made. A gem created by bundler:

* Does not create a dummy Rails application
* Does not declare a dependency on Rails (because you may just want a component of it)
* Does not include a test directory
  - Hydra uses Rspec so the test/ directory of Rails plugin is not needed

From this point I begin crafting my Rails plugin.

I make sure to add [rspec](https://github.com/rspec/rspec) as a development dependency.
Then I begin working on the plugin.

## Start with the Smallest Dependencies

In other words, skip putting `s.add_dependency "rails", "~> x.x"` in your gemspec file.
Instead focus on its constituent parts:

* Will you be exposing controllers, views, and/or routes? Then depend on `actionpack`
* Do you want some of the syntactic Rails sugar? Then depend on `activesupport`
* Do you want to include database models? Then depend on `activerecord`

Or maybe you don't need any of that. **Won't that be liberating.**

## Am I an Engine? Am I a Railtie?

A [Rails::Application](https://github.com/rails/rails/blob/master/railties/lib/rails/application.rb#L79) extends a [Rails::Engine](https://github.com/rails/rails/blob/master/railties/lib/rails/engine.rb#L337) which extends a [Rails::Railtie](https://github.com/rails/rails/blob/master/railties/lib/rails/railtie.rb#L114).

### Rails::Railtie

> Railtie is the core of the Rails framework and provides several hooks to extend Rails and/or modify the initialization process.
>
> An extension doing any of the following would require Railtie:
> * creating initializers
> * configuring a Rails framework for the application, like setting a generator
> * adding `config.*` keys to the environment
> * setting up a subscriber with ActiveSupport::Notifications
> * adding rake tasks
>
> Extracted from [Rails::Railtie documentation](https://github.com/rails/rails/blob/master/railties/lib/rails/railtie.rb#L8-L21)

### Rails::Engine

> `Rails::Engine` allows you to wrap a specific Rails application or subset of
> functionality and share it with other applications or within a larger packaged application.
> Since Rails 3.0, every `Rails::Application` is just an engine, which allows for simple
> feature and application sharing.
>
> Any `Rails::Engine` is also a `Rails::Railtie`, so the same
> methods (like `rake_tasks` and `generators`) and configuration
> options that are available in railties can also be used in engines.

> Allows you to wrap a specific Rails application or subset of functionality and share it with other applications or within a larger packaged application.
> Since Rails 3.0, every `Rails::Application` is just an engine, which allows for simple feature and application sharing.
>
> Extracted from [Rails::Engine documentation](https://github.com/rails/rails/blob/master/railties/lib/rails/railtie.rb#L8-L21)

### Advice

So, if your plugin is going to have application like behavior (i.e. routing HTTP requests to an end-point), then make it an Engine.
Otherwise a Railtie will suffice.

If your Railtie starts developing application-like behavior, just change it from a Railtie to an Engine.

## Isolated Focus

While I'm working on the plugin, I don't want to load up the entire Rails stack just to test a method that is independent of Rails.
In other words, I want to work on my plugin in isolation.

Below is an example of how I do that. This is a common pattern for Engines and Railties.

```ruby
require 'myplugin/engine' if defined?(Rails)
module MyPlugin
end
```

The Engine (or even Railtie) should be required if you have a Rails context.
In other words, when you are integrating your plugin with an application.

## Write More than One Spec Helper

Pay attention to the context of what you are testing.
Keep your dependencies in mind.
Isolated focus!

[Hydramata::Works](https://github.com/ndlib/hydramata-works/) has 4 context specific spec helpers:

* [./spec/spec_fast_helper.rb](https://github.com/ndlib/hydramata-works/blob/master/spec/spec_fast_helper.rb)
* [./spec/spec_active_record_helper.rb](https://github.com/ndlib/hydramata-works/blob/master/spec/spec_active_record_helper.rb)
* [./spec/spec_view_helper.rb](https://github.com/ndlib/hydramata-works/blob/master/spec/spec_view_helper.rb)
* [./spec/spec_slow_helper.rb](https://github.com/ndlib/hydramata-works/blob/master/spec/spec_slow_helper.rb)

Consider the following spec:

```ruby
require 'spec_fast_helper'
require 'hydramata-works'

module Hydramata
  describe Works do
    it 'has a .table_name_prefix to conform to Rails::Engine' do
      expect(described_class.table_name_prefix).to eq('hydramata_works_')
    end

    it 'has a .use_relative_model_naming? to conform to Rails::Engine' do
      expect(described_class.use_relative_model_naming?).to eq(true)
    end
  end
end
```

I run the following command:

```console
$ time rspec -r spec/<named_helper.rb> spec/lib/hydramata-work_spec.rb
```

| Helper                       | real     | user     | sys      |
|------------------------------|----------|----------|----------|
| spec_fast_helper.rb          | 0m1.423s | 0m1.276s | 0m0.142s |
| spec_active_record_helper.rb | 0m1.953s | 0m1.674s | 0m0.257s |
| spec_view_helper.rb          | 0m2.003s | 0m1.720s | 0m0.274s |
| spec_slow_helper.rb          | 0m3.854s | 0m3.166s | 0m0.661s |

The test times for the `spec_fast_helper`, `spec_active_record_helper`, and `spec_view_helper` grow as you:

* Add more gem dependencies
* Add more dependencies to the file you are testing (i.e. more require statements)

The test times for `spec_slow_helper` grow as you:

* Add more gem dependencies
* Add more dependencies to the file you are testing (i.e. more require statements)
* Add any ruby files to your plugin

Which means my `spec_slow_helper` has been getting slower and slower as the files in my project grows.
Other specs, because they are isolated, continue to run fast(ish).

**Note: The `spec_slow_helper` is a renaming of the conventional `spec_helper`.*

## Conclusion

This careful focus on crafting the plugin means:

* I only "require" what I need
* I keep my requirements explicit
* I have tests that run as fast as possible given their dependencies
* I stay outside of Rails as much as possible

This focus helps me rapidly iterate on a solution.

## Resources

* [Rails Guide to Plugins](http://guides.rubyonrails.org/plugins.html)
* [Rails Guide to Engines](http://guides.rubyonrails.org/engines.html)
  - Beware the `isolate_namespace` for it can cause grief
* [Isolate Namespace in Rails Engines - A Hack in Itself](http://crypt.codemancers.com/posts/2013-09-22-isolate-namespace-in-rails-engines/)
* [Orcid engine](http://github.com/projecthydra-labs/orcid)
* [Hydramata::Works engine](http://github.com/ndlib/hydramata-works)
* [Engine Cart](https://github.com/cbeer/engine_cart)
  - Note [pull request about removing depdency](https://github.com/cbeer/engine_cart/pull/17)
* [Engine Factory](https://github.com/projecthydra-labs/engine_factory)
