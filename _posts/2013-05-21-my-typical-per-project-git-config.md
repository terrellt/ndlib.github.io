---
author:   jeremyf
category: practices
filename: 2013-05-21-my-typical-per-project-git-config.md
layout:   post
tagline:  My typical .git config for projects for which I'm a contributor
title:    Project Hydra Git Config
tags:     git, hydra
---

I am a contributor to various Github projects and organizations:

* [Project Hydra](https://github.com/projecthydra)
* [Hesburgh Libraries of Notre Dame](https://github.com/ndlib)

Over the past year I've went from primarily working on my own projects, to collaborating on projects.
What I have found is that I need to normalize my Git configurations.

For each project (i.e. [Curate](https://github.com/ndlib/curate), [Sufia](https://github.com/projecthydra/curate), [ActiveFedora](https://github.com/projecthydra/active_fedora), etc.) I have converged on the following protocol:

    [remote "origin"]
      # This is a read-only URL; I can't accidentally push to it
      url = git://github.com/ndlib/curate.git
      fetch = +refs/heads/*:refs/remotes/origin/*
    [branch "master"]
      remote = origin
      merge = refs/heads/master
    [remote "mine"]
      # This is my branch; I'm pushing branches up to this remote for submitting pull requests
      url = git@github.com:jeremyf/curate.git
      fetch = +refs/heads/*:refs/remotes/mine/*
    [remote "__origin__"]
      # This is a a read/write URL for the remote branch; I can use it in a pinch
      url = git@github.com:ndlib/curate.git
      fetch = +refs/heads/*:refs/remotes/mine/*

The way this works for me is I regularly update my local repo from "origin" (via `git pull --rebase`).
I use my remote repo solely for submitting pull requests.

## Workflow

* Fork the original project on Github
* Clone the original project locally, using the read-only URL; This will be "origin"
* Add my fork as a remote via `git remote add mine <read-write-url-for-my-fork>; This will be "mine"
