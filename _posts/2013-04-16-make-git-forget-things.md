---
author:      danhorst
category:    practices
description: There are some things that you don't want to commit to source control.
filename:    2013-04-16-make-git-forget-things.md
layout:      post
tagline:     Sometimes you want git to ignore changes not files
tags:        git
title:       Make git look the other way
---

There are some things that you don't want to commit to source control.
A common case is configuration files where your local setup differs than the canonical one.
You can make `git` ignore entire files by adding them to `.gitignore` or `~/.gitignore`.
In the case of configuration files this isn't really helpful.

Fortunately you can make `git` ignore _changes_ you've made to a tracked file.
I have the following options set in my `~/.gitconfig`:

    [alias]
      forget = update-index --assume-unchanged
      remember = update-index --no-assume-unchanged

That way I can tell `git` to forget the changes I just made to my database configuration by:

    $ git forget config/database.yml

Then `config/database.yml` won't show up as being altered when I do `git status` or make a commit.

This is handy but not flawless.
For example, when you merge or rebase your `HEAD` your forgotten changes may disappear.
Sometimes I keep a local copy of the changed file in a untracked location like `tmp/config/database.yml`.
That way I can just copy it over after I merge or rebase.

Feel free to automate this if you like.
