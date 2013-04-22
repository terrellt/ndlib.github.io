---
author:   jeremyf
category: practices
filename: 2013-04-20-ruby-resources-for-the-university.md
layout:   post
tagline:  Diving into Ruby for the "First Time"
title:    Ruby Resources for the University
tags:     ruby
---

## Ruby Resources

### Definitions

* `irb` - interactive ruby command line
* `gem` - a Ruby library, packaged up for easy inclusion in other projects.
* `rails` - a web application framework that "put Ruby on the map"
* `bundler` - a `gem` dependency management system

### Links

* [Ruby Language Homepage](http://ruby-lang.org) - Homepage for Ruby
* [The Ruby Toolbox](http://www.ruby-toolbox.com) - Search for popular gems by topic
* [Bundler](http://gembundler.com) - "The best way to manage a Ruby application's gems"
* [Ruby on Rails](http://guides.rubyonrails.org/) - Ruby's flagship web application framework
* [RVM](https://rvm.io/) - manage multiple versions of ruby on your machine
* [rbenv](https://github.com/sstephenson/rbenv/) - a less "intrusive" alternate to RVM
* [Ruby Debugger](https://github.com/cldwalker/debugger) - a debugger for Ruby
* [A Non-Exhaustive List of Helpful Troubleshooting Things](https://github.com/ldcx/ldcx-2013/blob/master/sessions/tools-for-troubleshooting-ruby.md)

### Books

I would highly recommend the following Ruby and Rails books:

* ["Practical Object-Oriented Design in Ruby" by Sandi Metz](http//practicaloodinruby.com)
* ["Design Patterns in Ruby" by Russ Olsen](http://designpatternsinruby.com)
* ["Ruby Best Practices" by Gregory T Brown](http://www.amazon.com/Ruby-Best-Practices-Gregory-Brown/dp/0596523009/ref=sr_1_14?ie=UTF8&qid=1366481685&sr=8-14&keywords=ruby+programming)
* ["Agile Web Development with Rails" by Sam Ruby](http://pragprog.com/book/rails4/agile-web-development-with-rails)
* ["Rails Test Prescriptions: Keeping Your Application Healthy" by Noel Rappin](http://pragprog.com/book/nrtest/rails-test-prescriptions)
* ["Programming Ruby 1.9 (3rd Edition): The Pragmatic Programmer's Guide" by Dave Thomas with Chad Fowler and Andy Hunt](http://pragprog.com/book/ruby3/programming-ruby-1-9)

I haven't read this, but Francis may very well be interested in it:

* ["Build Awesome Command-Line Applications in Ruby: Control Your Computer, Simplify Your Life" by David Bryant Copeland](http://pragprog.com/book/dccar/build-awesome-command-line-applications-in-ruby)

### Considerations

The Ruby community is traditionally focused on automated testing.
Commit to writing tests.

You will:

* write better code.
* gain a deeper understanding of your code.
* express what your code is doing.
* better be able to adapt to changes.
* be able to reduce the chance of regressions.

And paradoxically, you will develop skills to write software solutions faster.
Also be mindful that if it is hard to test, it is quite possible that it will be *very* hard to work with.

Think of your tests as an important letter to a future developer.
That future developer may be you; or the new developer joining your team.
This letter should be as clear and concise as possible.

Your test suite is just as valuable as your production code, perhaps more so.
It verifies that what you have written for production actually works.
It can provide reassurances as you upgrade your software.

There are several types of tests that you could end up writing:

* `Unit tests` - testing a singular class with little to no interaction with other classes
* `Functional tests` - testing the interaction of a set of closely related objects
* `Integration tests` - testing the systems behavior with minimal knowledge of its underworkings

#### Testing Resources

* [Capybara](https://github.com/jnicklas/capybara) - "helps you test web applications by simulating how a real user would
interact with your app."
* [WebMock](https://github.com/bblimke/webmock) - "Library for stubbing and setting expectations on HTTP requests in Ruby."
* [Timecop](http://github.com/travisjeffery/timecop) - "A gem providing "time travel" and "time freezing" capabilities"
* [Guard](https://github.com/guard/guard) - "A command line tool to easily handle events on file system modifications"

## Tasks at Hand

Now with all that out of the way, on to the tasks.

### Search LDAP

From [Ruby Net::LDAP](https://github.com/ruby-ldap/ruby-net-ldap)

    require 'rubygems'
    require 'net/ldap'

    ldap = Net::LDAP.new :host => server_ip_address,
         :port => 389,
         :auth => {
               :method => :simple,
               :username => "cn=manager, dc=example, dc=com",
               :password => "opensesame"
         }

    filter = Net::LDAP::Filter.eq("cn", "George*")
    treebase = "dc=example, dc=com"

    ldap.search(:base => treebase, :filter => filter) do |entry|
      puts "DN: #{entry.dn}"
      entry.each do |attribute, values|
        puts "   #{attribute}:"
        values.each do |value|
          puts "      --->#{value}"
        end
      end
    end


### Create a file

File from Ruby Standard Lib

    # Write "Hello World" to the specified path:
    File.open("./path/to/my/file.txt", 'wb') { |file|
      file.puts "Hello World"
    }

Write a CSV file from Ruby Standard Lib (Ruby 1.9.3 or greater)

    CSV.open("path/to/file.csv", "wb") do |csv|
      csv << ["row", "of", "CSV", "data"]
      csv << ["another", "row"]
      # ...
    end

### SSH/SCP a file to the server

Invoke command line methods

    `scp ./path/to/my/file my.server.net:/path/to/destination`

From [Net::SSH](http://github.com/net-ssh/net-ssh)

    require 'net/ssh'

    Net::SSH.start('host', 'user', :password => "password") do |ssh|
      # capture all stderr and stdout output from a remote process
      output = ssh.exec!("hostname")

      # capture only stdout matching a particular pattern
      stdout = ""
      ssh.exec!("ls -l /home/jamis") do |channel, stream, data|
        stdout << data if stream == :stdout
      end
      puts stdout

      # run multiple processes in parallel to completion
      ssh.exec "sed ..."
      ssh.exec "awk ..."
      ssh.exec "rm -rf ..."
      ssh.loop

      # open a new channel and configure a minimal set of callbacks, then run
      # the event loop until the channel finishes (closes)
      channel = ssh.open_channel do |ch|
        ch.exec "/usr/local/bin/ruby /path/to/file.rb" do |ch, success|
          raise "could not execute command" unless success

          # "on_data" is called when the process writes something to stdout
          ch.on_data do |c, data|
            $STDOUT.print data
          end

          # "on_extended_data" is called when the process writes something to stderr
          ch.on_extended_data do |c, type, data|
            $STDERR.print data
          end

          ch.on_close { puts "done!" }
        end
      end

      channel.wait

      # forward connections on local port 1234 to port 80 of www.capify.org
      ssh.forward.local(1234, "www.capify.org", 80)
      ssh.loop { true }
    end


### Simple web-app

[Ruby on Rails](http://guides.rubyonrails.org)
[Getting started](http://guides.rubyonrails.org/getting_started.html)

#### ...to query a database and return records

    # Creates a new rails application, using sqlite
    $ rails new APP_PATH

    # Creates a new rails application, using postgresql
    $ rails new APP_PATH -d postgresql

    # Use the scaffold to get kick the tires
    $ cd APP_PATH
    $ rails generate scaffold Blog title:string content:text

#### ...to return a JSON response

    class BlogController
      def index
        @blogs = Blog.all
        respond_to do |format|
          format.html
          format.json # Add this line
        end
      end
    end