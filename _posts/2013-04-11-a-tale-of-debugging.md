---
author:   jeremyf
category: practices
filename: 2013-04-11-a-tale-of-debugging.md
layout:   post
tagline:  In which our blogger discovers he was his own worst enemy
title:    A Tale of Debugging
tags:     rails
---

I am in the process of extracting [CurateND – an existing application](https://curate.nd.edu) – into the [Curate gem – a Rails engine](https://github.com/ndlib/curate).

## Framing the Problem

During this process I stumbled upon a ridiculously vexing problem.
Some of my Engine's controller tests were failing, saying that an an object created by the current user couldn't be seen by the current user.
I stepped down into an CanCan::Ability test to see if this was the problem, but those tests passed.

Which meant, *something was rotten in Controller land*.

With a bit more digging, I discovered that the current user wasn't getting set.
Ultimately I wrote an automated test to start testing against.
Below is the initially failing test that I wrote:

```
  describe ApplicationController do
    let(:user) { FactoryGirl.create(:user) }
    it 'should set :current_user on signin' do
      expect {
        sign_in(user)
      }.to change(controller, :current_user).from(nil).to(user)
    end
  end
```

All told, I spent about 6 hours troubleshooting this.
Had I not spent the week dealing with load dependencies of an Engine that uses an Engine, I think I may have stumbled upon the solution much earlier.
But my mind was already looking for one kind of problem, so when this one showed up, I assumed it was similar.

## Steps for Solving the Problem

I went to the [ruby debugger gem](http://rubygems.org/gems/debugger) and began stepping into the code.
Stepping through, I realized that I was going to need to insert breakpoints at numerous places if I was going to gain an understanding of the implementation details of Devise and Warden.
And this is where I grabbed [my new favorite gem: method_locator](http://rubygems.org/gems/method_locator).

To my gem's Gemfile I added the following:

```
  gem 'method_locator'
```

Then in my debugger console, I could do the following to get where all the method was defined:

```
(rb:1)$ e methods_for(:sign_in).collect{|m| m.source_location.join(':') }
=> ["/path/to/gems/ruby-2.0.0-p0@curate/gems/devise-2.2.3/lib/devise/test_helpers.rb:45"]
```

*If :sign_in was overridden in a descendant class, the above array would've had two or more entries.*
*One for each definition.*
*Which is the long way of saying, I could see where `super` was being used.*

Now with the path to the method I could:

* open the file in my editor
* add a runtime break point (as below)

```
(rb:1)$ break /path/to/gems/ruby-2.0.0-p0@curate/gems/devise-2.2.3/lib/devise/test_helpers.rb:45
```

From there I could debug away, stepping into the bowels of Warden and Devise.
And what did I learn?
First, those are two beautiful code sets that successfully and elegantly interact.
Second, in attempting to take a shortcut, I had in fact created a whole lot of work.

Below is the class that proved to problematic, and it was written by me earlier this week.

```
class User < ActiveRecord::Base
  include Sufia::User
  devise :recoverable, :database_authenticatable, :registerable,
    :recoverable, :rememberable, :trackable, :validatable

  def password; 'password'; end

  def encrypted_password
    password_digest(password)
  end
end
```
*Hint: From the debugger, I could do `User.new.encrypted_password` multiple times, and get different answers.*

To be fair, the class was in my dummy application, and I just wanted the bare metal to get it working.
So when I was receiving NoMethodError exceptions on User#encrypted_password, I simply defined it and moved on.

Ultimately, the problem was that a portion of the user's encrypted password was being used as the :authenticatable_salt portion of the session.
And my user's encrypted password was changing over time.

Take a look at Github for [the commit that fixed the problem](https://github.com/ndlib/curate/commit/97ab9c71354e35bf9f3e97b748f856833b952153).