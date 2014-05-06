---
author:   jeremyf
category: practices
filename: 2014-05-05-be-mindful-of-your-gemspec-s-bin-declaration.md
layout:   post
tagline:  Because bin/rails will mess you up
title:    Be Mindful of Your Gemspec's Bin Declaration
tags:     ruby, rails
---

## TL;DR Version

Be mindful of what you add to your gemspec's executables.
If you add `bin/rails` expect [trouble](https://github.com/cbeer/engine_cart/issues/9).

## The Long Version

At [LDCX^5](http://library.stanford.edu/projects/ldcx/2014-conference), some [ProjectHydra developers](https://github.com/projecthydra-labs/engine_factory/graphs/contributors) went through the ritual of creating a template for creating namespaced Rails engines.

First we went through the steps to create a namespaced rails plugin.
Then we codified that via a [Rails plugin template](https://github.com/projecthydra-labs/engine_factory/blob/master/lib/template.rb).

### Gem Creation

When creating a gem I usually opt for `bundle gem <name_of_gem>`.
However, we wanted to leverage the behavior for generating a Rails plugin (e.g. `rails plugin new <name_of_gem>`).

Compare the `rails plugin new <name_of_gem>` gemspec:

```ruby
s.files = Dir["{app,config,db,lib}/**/*", "MIT-LICENSE", "Rakefile", "README.rdoc"]
s.test_files = Dir["test/**/*"]
```

To the `bundle gem <name_of_gem>`:

```ruby
spec.files         = `git ls-files -z`.split("\x0")
spec.executables   = s.files.grep(/^bin\//) { |f| File.basename(f) }
spec.test_files    = s.files.grep(/^(test|spec|features)\//)
spec.require_paths = ['lib']
```

I prefer the bundler syntax as it ensures we get most everything from the gem.

### Enter the Odd Duck

What ensued was weird behavior that [Engine Cart](https://github.com/cbeer/engine_cart) exposed.
Namely, my `rails` command was being overwritten.
Below is a pristine `rails` command.

```ruby
require 'rubygems'

version = ">= 0"

if ARGV.first
  str = ARGV.first
  str = str.dup.force_encoding("BINARY") if str.respond_to? :force_encoding
  if str =~ /\A_(.*)_\z/
    version = $1
    ARGV.shift
  end
end

gem 'railties', version
load Gem.bin_path('railties', 'rails', version)
```

After engine_cart ran its [`bundle install` command](https://github.com/cbeer/engine_cart/blob/v0.3.4/lib/engine_cart/tasks/engine_cart.rake#L73) against the interal application, my `rails` command was overwritten, replacing "railties" with "name_of_gem".

```ruby
require 'rubygems'

version = ">= 0"

if ARGV.first
  str = ARGV.first
  str = str.dup.force_encoding("BINARY") if str.respond_to? :force_encoding
  if str =~ /\A_(.*)_\z/
    version = $1
    ARGV.shift
  end
end

gem 'name_of_gem', version
load Gem.bin_path('name_of_gem', 'rails', version)
```

As you might guess, this creates lots of problems.
So I modified my gemspec to the following:

### The Kludgy Solution

```ruby
s.files         = `git ls-files -z`.split("\x0")
s.executables   = s.executables   = s.files.grep(%r{^bin/}) do |f|
  f == 'bin/rails' ? nil : File.basename(f)
end.compact
s.test_files    = s.files.grep(/^(test|spec|features)\//)
s.require_paths = ['lib']
```

The solution is not elegant, but it appears to get the job done.