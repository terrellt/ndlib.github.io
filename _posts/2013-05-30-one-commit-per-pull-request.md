---
author:   jeremyf
category: practices
filename: 2013-05-30-one-commit-per-pull-request.md
layout:   post
tagline:  How to take the pain out of managing pull requests
title:    One Commit per Pull Request
tags:     git, github
---

I've been contributing to more projects in which it is expected that I should submit a [pull request that contains a single commit](https://github.com/projecthydra/hydra-head/blob/master/CONTRIBUTING.md).
Personally, I've adopted the _commit early and commit often_ mentality for development.
That means, I make tiny atomic commits as I work towards the solution.

* [Jump to git rebase --interactive](#git-rebase-interactive)
* [Jump to .gitconfig for `git squash` command](#git-squash)

The _commit early and commit often_ for local development is in opposition to the required single commit for a pull request.

For pull requests, a single commit is easier to inspect, critique, and discuss.
By creating a single commit, I am saying "This is a logical unit of work" for the project.
I can explain what and why these changes are made - developer documentation if you will.
At a future date, for other contributors, it is easier to get a context for a small change in one file if that change tracks to a larger unit of work.

For local development, the multiple commit methodology is me iterating towards a solution.
I can divide the overall solution into chunks and work on those.
Then when a chunk is done, I can commit the change locally.

## My Local Repository

Locally, on my `development-service-object` branch, I may have the following commits:

    * 4d83d01 — Making ServiceObject work for new case Jeremy Friesen, (1 hour ago)
    * 3d87c01 — Adjusting ServiceObject API for new case Jeremy Friesen, (2 hours ago)
    * c911583 — Making ServiceObject do the work Jeremy Friesen, (3 hours ago)
    * fc7bad8 — Creating ServiceObject Jeremy Friesen, (4 hours ago)

Each of these commits represent a unit of work on my machine.
It is very likely that latest and third latest commit are working directly on the ServiceObject.

When I submit a pull request, I want to have one commit.
And I want to make sure the commit message conveys the changes.

## git rebase --interactive
<span id="git-rebase-interactive"></span>

I want to squash those commits into a single commit so I can submit a pull request.
To do this I first make sure my repository is up to date:

    $ git checkout master
    $ git pull --rebase canonical # The remote repo that is "canonical"
    $ git checkout development-service-object
    $ git rebase master

I'll make sure the tests pass, then run the following command: `git rebase --interactive HEAD~4` (4 because I have four commits for this pull request)

And get the following interactive menu:

    pick fc7bad8 Creating ServiceObject
    pick c911583 Making ServiceObject do the work
    pick 3d87c01 Adjusting ServiceObject API for new case
    pick 4d83d01 Making ServiceObject work for new case

I make the following updates and save the change:

    pick fc7bad8 Creating ServiceObject
    squash c911583 Making ServiceObject do the work
    squash 3d87c01 Adjusting ServiceObject API for new case
    squash 4d83d01 Making ServiceObject work for new case

And get the following default commit message to rewrite:

    # This is a combination of 4 commits.
    # The first commit's message is:
    Creating ServiceObject

    # This is the 2nd commit message:

    Making ServiceObject do the work

    # This is the 3rd commit message:

    Adjusting ServiceObject API for new case

    # This is the 4th commit message:

    Making ServiceObject work for new case

At this point, I completely and entirely rework my commit message as per the [contributing guidelines](https://github.com/projecthydra/hydra-head/blob/master/CONTRIBUTING.md) of the project.

Then I push the `development-service-object` branch to my remote repository, and issue a pull request to the "canonical" remote repository.

## git squash
<span id="git-squash"></span>

Early on, one of the challenges for squashing commits was knowing how many commits I needed to squash.

The following command works:

    $ git log master..development-service-object --oneline | wc -l | tr -d " "
    > 4

I like leveraging git aliases, so I wrote up a few helpful git aliases and put them in my `~/.gitconfig` file:

    [alias]
      branch-current = rev-parse --symbolic-full-name --abbrev-ref HEAD
      number-of-commits-since-master = "! sh -c 'git log master..`git branch-current` --oneline | wc -l | tr -d \" \"'"
      squash = "! sh -c 'git rebase --interactive HEAD~`git number-of-commits-since-master`'"

It should be noted that this works for my use case, but there may be edge cases or differing repository states that this may not work.
