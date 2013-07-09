---
author:   danhorst
category: planning
filename: 2013-01-03-a-serviceable-digital-repository.md
layout:   post
reblog:   'http://www3.nd.edu/~dbrubak1/planning/a-serviceable-digital-repository/'
tagline:  Building A Digital Repository We Can Live With
title:    A Serviceable IDR
---

## Setting Expectations

The primary deliverable of a digital repository should be a persistent URL that resolves to a rich HTML description of an item.
A digital object should be able to be:

 - Linked to
 - Embedded (e.g. via [oEmbed][1])
 - Indexed by Search Engines (e.g. Google, Google Scholar)
 - Presented in our [Library Catalog][2]
 - Delivered in a wide variety of derivatives

All digital objects need:

 - Access controls
 - Copyright auditing
 - Life cycle management
 - Description and metadata enrichment

We need to provide the services needed to meet these expectations when the digital objects are ingested.

When we store digital objects the original files will be treated specially.
In most cases the original file will be a preservation ready item.
We will want to create a presentation-ready derivative of the original at a reasonable fidelity and in an appropriate format.
When preserving these files the archival and the presentation masters of the file should be versioned and periodically bit-checked for integrity.
On-the-fly derivatives, such as a crop from an image or a plain text export of a TEI document, do not need integrity checking.
Derivatives should be cached and allowed to expire via an appropriate mechanism.


## Don't Dig The Hole Deeper

While the Hydra stack has been helpful it is not without problems.
Thus far our efforts have produced applications that are complicated, difficult to maintain, and fail at delivering an exceptional experience to patrons and curators.
Penn State's [ScholarSphere][3] is a well-implemented "Hydra head" but it is still constrained by the same design decisions present in all "Hydra heads".
Applications that use the "Hydra head" design philosophy have awkwardly coupled concerns.
Persistence, serialization, and presentation are blended together and the resulting application is entwined with Fedora, Solr, and a host of gems.

The promise of the Hydra stack is that if you install a bunch of gems into your application it will be easy to make a Fedora-backed digital repository.
This is fine if there is only one application that serves as a digital repository for an institution.
With each added application it gets more and more complex to maintain the ecosystem because the necessary gems are rapidly changing.
We already have a half dozen applications and are on track to build at least that many this year.
I think this will create a maintenance nightmare.


## An API-Driven Architecture

I believe that we can forestall our growing pains if we reexamine our architecture.
The collection of gems that makes up a "Hydra head", which as also been colloquially referred to as the "Hydra neck", isn't a good integration point across multiple applications because it ties every application too closely to Fedora and Solr.

Right now when we say "repository" we mean Fedora Commons.
When we say "repository" we _should_ mean a common API for storing and retrieving digital objects.
If we wrap Fedora and the canonical Solr index with an application that provides a [Hypermedia API][4] to the underlying content we can keep a lot of the complexity centralized.
This will effectively move the point of integration between our applications from the Fedora API to the new API that we have written to meet our needs.
This should lower the barrier of implementation for client applications after an upfront investment in time.

One of the topics that perennially pops up at Hydra meetings is: "What would Hydra look like without Fedora?"
The propsed API-driven architecture offers an answer.
Because it removes the Fedora API as the common interface between applications the entire persistence layer can be changed to anything that implements the API.
This also provides a benefit for developers because it would allow alternate implementations of the API, e.g. one backed by SQLite, to be used while developing repository-backed applications.


## Patron-Facing API Clients

While the centralized repository will have a canonical representation of each object most of the time an object belongs to a more specific context.
Groups of digital objects should be able to be embedded or re-cast in other interfaces as members of a collection, exhibit, or other specialized interfaces.

### Exhibit
An exhibit is a selection of items with particular scholarly significance.
They should stand alone or serve as digital companions to exhibits presented in the Rare Books exhibit area.
Exhibits may be as simple as static pages with embedded repository content.
There is room for a tool like Atrium to facilitate this process but it's present incarnation is more focused on "collections" than "exhibits".

### Collection
A collection interface is an alternative to a conventional finding aid.
Collections will typically be based on Blacklight and have their own collection-specific Solr indexes.

### Portal
A collection with additional features and visualizations.
We can provide value to our patrons with tools that enable them to sift, sort, and juxtapose our holdings.
The collection interface would typically be Blacklight-driven while the other components may be either shared visualization tools or one-off pieces as needed.


## Selector-Facing API Clients

One of the selling points of Hydra is that it can unite the patron-facing discovery interface with the curator-facing administrative interface within the same application.
While this is nice in some cases, such as in a self-submit style application, the needs of a patron and a curator can diverge dramatically in some cases.
In the Seaside project we split the administrative interface and the discovery interface into two applications.
This allows us to taylor the experince for each audience.
We can extend this idea further and create a variety of tools for adding and editing content into the repository to meet the needs of curators in different contexts.

### Single-item Editor
This can be accomplished in a traditional Hydra-style form view in an application.
To enable editing items from the front-end view we could create a simple browser extension, or perhaps even a bookmarklet, that embeds an editing interface onto the presentation page.
This would work best when there are only a few curators.

### Batch Ingest
There are a number of areas that use Filemaker Pro or other databases to manage their collections.
Thick clients are sometimes very helpful and we shouldn't force them to abandoned their established workflow to accommodate our repository.
Hopefully writing an adapter from FMP using a client to our repository API will be simpler than our current solution of writing rake tasks that call ActiveFedora directly.

### Depository
In a depository privileged patrons can curate their own submissions.
In this case a conventional "Hydra head" like [Libra][5] and [ScholarSphere][3] would be a fine solution.


## Next Steps

There is a lot of work involved in this proposal.
Here are some of the deliverables:

- Determine if adding another layer of abstraction in our infrastructure will result in a net loss of work.
I believe it has the potential to do so but I have no empirical data.

- Design the one repository API to rule them all.
This will not be easy.
There are a lot of things to get wrong and arm waving about [HATEOS][6] doesn't make the technical challenges disappear.

- Implement the API.
If we do the design work up front this will easier but it is still a major project.

- Implement the repository API with a different persistence mechanism.
It would greatly simplify the development stack if the API could be implemented on top of something like Redis or SQLite.

- Write clients for the API.
A Ruby client and a JavaScript client would be a good start.

- Determine how to make repository content portable.
[oEmbed][1] is one option; are there others?


[1]: http://oembed.com/
[2]: http://onesearch.library.nd.edu/primo_library/libweb/action/search.do?vid=NDU
[3]: https://scholarsphere.psu.edu/
[4]: http://blog.steveklabnik.com/posts/2012-02-27-hypermedia-api-reading-list
[5]: http://libra.virginia.edu/
[6]: http://en.wikipedia.org/wiki/HATEOAS
