---
author:   jeremyf
category: practices
filename: 2014-04-28-addressing-a-primary-concern-with-the-ui-based-deposit.md
layout:   post
tagline:  Perhaps the UI is too naive. And should be treated like we would a batch?
title:    Addressing a Primary Concern with the UI-based deposit
tags:     ruby, rails
---

Consider the following implicit acceptance test for a user's deposit:

```gherkin
Given I am a uploading a set of files and their data.
When the upload is marked "invalid" for some reason.
Then none of the files should be in Fedora
And none of the data should be in Fedora
```

In the present situation a single failure late in the "save" sequence of events results in the data pre-failure data being in Fedora and SOLR;
But an error reported to the UI saying there was a problem.
In other words, a failure late in the save sequence means that there are some files in Fedora, but the user was told that nothing was saved.

This may be obviated by the transactions of Fedora 4, but do we want our users waiting for the response cycle to see if:

* All of the validations passed
* All of the data was sent to Fedora and SOLR
* All of the files were sent to Fedora and SOLR

This means the user must wait for all of the HTTP requests that are made against Fedora.
Is this acceptable?

We can know if the data is valid before hand, why should we make the end user wait for us to push everything into Fedora.
They told us what they want to send.
Why not sequester it?

## A Proposed Solution

Consider the following simple POST request: `https://curate.nd.edu/concern/article?title=bob&file=path/to/file`

* Parse Request - deal with the routes and parameters
* Verify Authorization - can the user do what they are asking
* Validate Request - is the input valid for the given user?
* Persist (see below)
* Issue response
  - User facing messaging - flash messages ("Thank you for your deposit. Your files are being worked on right now. They won't be available for searching, but you can see them 'here'.")
  - HTTP response - redirect, rendered template, HTTP status, etc.

### Persist

* Persist raw parameters and files to simple storage document.
  * *We said everything was valid so its now in our hands.*
  * Preserve user context (i.e. who did what)
  * Fire off ingest job, responsible for (but invariably configurable):
    * Running anti-virus
    * File characterization
    * Derivative generation
    * Piece-meal posting to Fedora; Since Fedora 3 is non-transactional, we need to do this iteratively.
    * Once you are done posting to Fedora, begin posting to SOLR; Issuing one commit per post.
    * Tidy up simple storage document
    * Notify submitter (and others) that ingest has completed

### Ingest Job

While the ingest job is running, the object can be retrieved from the Simple Storage document.
Until it is posted to SOLR, the file cannot be found via search.

We end up with three representations of the information:

* SOLR document
* Fedora document
* Simple Storage document

Each serves a purpose, and each addresses the complications of working with complex objects via multiple services.
