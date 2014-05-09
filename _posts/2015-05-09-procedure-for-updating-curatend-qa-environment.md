---
author:   jeremyf
category: procedures
filename: 2015-05-09-procedure-for-updating-curatend-qa-environment.md
layout:   post
tagline:  Applicable for 2014-Q2 CurateND Release
title:    Procedure for updating CurateND QA Environment
tags:     deploy
---

## Steps to Refresh the Staging/QA Environment

We are presently using `libvirt8.library.nd.edu` as our quality assurance/staging machine.
The target audience for this blog post is our internal team.
However you may find some high level ideas helpful.

First, understand the branching strategy.
More on [the branching strategy for releasing CurateND](http://ndlib.github.io/practices/leveraging-git-in-iterating-towards-a-release/)

## Make sure that CurateND's code is fresh:

1. In CurateND's `curatend-deploy-2014-04-29` branch
  1. `bundle update curate`
  1. commit those changes
  1. push those changes to CurateND repository
1. In `curatend-with-vanilla-data` branch
  1. `git rebase curatend-deploy-2014-04-29`
  1. `git push -f` those changes to CurateND repository

## Update the Data and Production Code

**ACHTUNG:** Pay attention to the `deploy_tag` parameter.
Due to the constraints of our Jenkin's implementation, [Capistrano](http://capistranorb.com/) tasks must be run from the `master` branch.
Which means these maintenance tasks are on our `master` branch.

1. Rebuild the Fedora, SOLR, application database
  * [Jenkin's Task](https://jenkins.library.nd.edu/jenkins/job/CurateND-FedoraDB-Rebuild/build?delay=0sec)
    * **host:** libvirt8.library.nd.edu
1. Migrate the application database to reflect
  * [Jenkin's Task](https://jenkins.library.nd.edu/jenkins/job/CurateND-STANDALONE/build?delay=0sec)
    * **cap_task:** maintenance:vanilla_to_nd_schema
    * **host:** libvirt8
    * **deploy_tag:** master
1. Migrate the work data structures
  * [Jenkin's Task](https://jenkins.library.nd.edu/jenkins/job/CurateND-STANDALONE/build?delay=0sec)
    * **cap_task:** maintenance:migrate_metadata
    * **host:** libvirt8
    * **deploy_tag:** master
1. Deploy the application changes
  * [Jenkin's Task](https://jenkins.library.nd.edu/jenkins/job/CurateND-STANDALONE/build?delay=0sec)
    * **cap_task:** deploy
    * **host:** libvirt8
    * **deploy_tag:** curatend-with-vanilla-data

## Go visit [libvirt8.library.nd.edu](https://libvirt8.library.nd.edu)

Now go and kick the tires.
If you find problems, report and capture those.
It may require a updating the code or refreshing the data.