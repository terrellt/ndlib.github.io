---
author:   jeremyf
category: practices
filename: 2014-05-09-leveraging-git-in-iterating-towards-a-release.md
layout:   post
tagline:  Its all about atomic commits and branches
title:    Leveraging Git in Iterating Towards a Release
tags:     git
---

**[Sibling Blog Post regarding Deployment](/procedures/2015-05-09-procedure-for-updating-curatend-qa-environment.md)**

We are in the process of deploying changes from the [Curate Engine](https://github.com/projecthydra-labs/curate) to our [CurateND application](https://curate.nd.edu).
This has involved a multi-step git coordination.

## Handling Custom Modifications to Curate Engine

We needed to make custom modifications of the Curate Engine.
So we:

* [forked Curate](https://github.com/ndlib/curate) to the [Hesburgh Libraries of Notre Dame organization](https://github.com/ndlib).
* [created a branch](https://github.com/ndlib/curate/tree/update-article-metadata) for localized changes

I did some local git configuration changes:

* `git remote set-url --push origin "Thou shalt not push"`
* `git remote add ndlib https://github.com/ndlib/curate.git`

Below is the resulting `.git/config`

```git
[remote "origin"]
  url = git@github.com:projecthydra/curate.git
  fetch = +refs/heads/*:refs/remotes/origin/*
  pushurl = Thou shalt not push
[remote "ndlib"]
  url = https://github.com/ndlib/curate.git
  fetch = +refs/heads/*:refs/remotes/ndlib/*
```

Then I pushed my new branch to ndlib (`git push ndlib branch-name`).

**Sidebar:** *By setting the push url for origin to "Though shalt not push", I can no longer  push changes to the projecthydra-labs/curate repository.*

## Making Modifications to CurateND

The next step was to update CurateND's reference to Curate.

* I made a branch in CurateND (eg `curatend-deploy-2014-04-29`).
* I updated the CurateND Gemfile to reference the new Curate version.

```ruby
gem 'curate', github: 'ndlib/curate', branch: 'update-article-metadata'
# gem 'curate', path: '../curate' # Sometimes I want the local reference for
#                                 # but we deploy the remote branch
```

On this branch we did what we needed to get the application working.

## Using Curate Reference Implementation Data

For this project, [Notre Dame is hosting a reference implementation](https://curatevanilla.library.nd.edu).
We acknowledged that for the QA process, we wanted to use a small subset of data.
The reason for the small set of data was that we were planning to regenerate the QA data as we iterated on the solution.

So we opted to grab all of the data from the reference implementation and copy that to a QA application.

This required copying:

* Fedora Commons data
* SOLR index
* Application's mySQL database schema and data

And here I encountered an annoying pair of gotchas.
Our reference implmenetation had three differences between CurateND.

1. The `users` table in mySQL was different.
   CurateND uses CAS whereas the reference implementation uses a local database-backed registration model.
   **We didn't want to drag our partners through the process of getting institutional credentials.**
1. The expected work types were different.
   CurateND has no concept of a GenericWork work type.
1. The Fedora namespace of the objects were different. (eg. 'und' vs. 'sufia')

## Handling the Difference

To resolve this, I created another branch off of `curatend-deploy-2014-04-29` (eg `curatend-with-vanilla-data`).

This new branch's purpose was to contain the necessary changes for addressing the three gotchas from above.
I then squashed those changes into one commit.

## Conclusion

I created the various forks and branchs to separate concernsk.

We needed isolate:

1. localized changes to Curate.
1. changes to CurateND to get Curate working.
1. modifications for using reference implementation data with production-oriented code-base.

In keeping these concerns as separate branchs, I've been able to make sure that I can:

1. grab Curate changes from upstream
1. modify production-oriented code without fear of mixing in the reference implementation
1. iterate through what is needed to allow access of the reference data via production code

I've also kept an eye on isolating commits so that I can push Curate changes upstream via pull requests.
