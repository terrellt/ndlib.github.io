---
author:   jeremyf
category: practices
filename: 2014-09-25-exposing-configuration-options-in-rails-engines.md
layout:   post
tagline:  Because Configuration May Need to Stomp on Convention
title:    Exposing Configuration Options in Rails Engines
tags:     rails, configuration
---

A common pattern for Rails Engines is to expose a `configure` method.
This blog post has three points of guidance regarding configuration:

* [Communicate Expectations for Configuration](#communicate-expectations-for-configuration)
* [Capture Configuration for Later](#capture-configuration-for-later)
* [Expose Configuration for Extension](#expose-configuration-for-extension)

First the basics for configuration.
Below is a common pattern for exposing configuration.

```ruby
module MyEngine
  class Configuration
    attr_accessor :configurable_service
  end
  class << self
    attr_writer :configuration
  end

  module_function
  def configuration
    @configuration ||= Configuration.new
  end

  def configure
    yield(configuration)
  end
end
```

Then in the Rails applications `./config/initializers` directory I add the following to the `my_engine_config.rb` file:

```
MyEngine.configure do |config|
  config.configurable_service = 'My Configuration'
end
```

Later in the application code, I can call `MyEngine.configuration.configurable_service`.
This works but I'm leaving out an important piece, what should `:configurable_service` be doing?

While working on [several](http://github.com/ndlib/hydramata-core) [Rails](http://github.com/ndlib/hydramata-works) [engines](http://github.com/projecthydra-labs/orcid), I have refined my configuration techniques.

<a name="communicate-expectations-for-configuration"></a>
## Communicate Expectations for Configuration

```
module MyEngine
  class Configuration
    class ConfigurationError < RuntimeError
    end

    def configurable_service
      @configurable_service ||= default_configuration_service
    end

    def configurable_service=(callable)
      if callable.respond_to?(:call)
        @configurable_service = callable
      else
        raise ConfigurationError.new("Expected #{callable.inspect} to respond_to #call. Could not set #{self.class}#configurable_service."")
      end
    end

    private
    def default_configuration_service
      lambda {|args| ... }
    end
  end
emd
```

This helps clarify and enforce expectations.
A few tests can help provide specific guidance as well.

<a name="capture-configuration-for-later"></a>
## Capture Configuration for Later

In some cases, I have encountered timing issues related to the Rails initializer sequence.
The typical problem is Rails raising a `NameError` or `LoadError` exception.
It is frustrating.

The solution that I've used is to leverage the [Rails::Railtie::Configuration#to_prepare method](http://api.rubyonrails.org/classes/Rails/Railtie/Configuration.html#method-i-to_prepare) (*Hint: The documentation is sparse.*).

Here are the changes I've made to work around the Initializer sequence problem. These are modified from the [Hydramata::Core gem](https://github.com/ndlib/hydramata-core) that I've worked on.

```ruby
module MyEngine
  if defined?(Rails)
    require 'rails/railtie'
    class Railtie < Rails::Railtie
      config.to_prepare do
        # Because I want allow for other components to plug into Hydramata's
        # configuration. Maybe there is a better way to do this.
        MyEngine.configure!
      end
    end
  end

  class Configuration
    attr_accessor :configurable_service
  end

  class << self
    attr_writer :configuration
  end

  module_function
  def configure(&block)
    @configuration_block = block
    configure! unless defined?(Rails)
  end

  def configure!
    if @configuration_block.respond_to?(:call)
      @configuration_block.call(configuration)
      @configuration_block = nil
    end
  end
end
```

The configuration block is captured as part of the initializer sequences, but is called just before the :after_initialize callback.

<a name="expose-configuration-for-extension"></a>
## Expose Configuration for Extension

Again, working on a suite of gems that are interrelated, I have found that I want a common configuration point.

```ruby
require 'active_support/lazy_load_hooks'
module MyEngine
  class Configuration
  end
  ActiveSupport.run_load_hooks(:my_engine_configuration, Configuration)
end

module AnotherEngine
  module ConfigurationMethods
    def pid_minting_service
      @pid_minting_service ||= default_pid_minting_service
    end

    def pid_minting_service=(callable)
      if callable.respond_to?(:call)
        @pid_minting_service = callable
      else
        raise RuntimeError, "Expected #{callable.inspect} to respond_to :call"
      end
    end

    private
    def default_pid_minting_service
      -> { "opaque:#{rand(100_000_000)}" }
    end
  end

  if defined?(Rails)
    class Engine < ::Rails::Engine
      initializer 'another_engine.configuration' do |app|
        ActiveSupport.on_load(:my_engine_configuration) do
          include ConfigurationMethods
        end
      end
    end
  end
end
```

The above will add the #pid_minting_service and #pid_minting_service= methods to MyEngine::Configuration.

## Conclusion

Configuration is powerful.
It is something that requires ample guidance to understand how to do it.

I find that exposing configurable function is one of the more powerful tools.
Maybe I don't always want to:

* Run a full blown anti-virus scan when I upload a file
* Perform expensive image transformations when I receive a file
* Perform complicated characterization of received files
* Call a remote service that requires I am always connected to the internet