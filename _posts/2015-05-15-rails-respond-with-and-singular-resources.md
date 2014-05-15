---
author:   jeremyf
category: practices
filename: 2015-05-15-rails-respond-with-and-singular-resources.md
layout:   post
tagline:  Surprise! Let me format that for you.
title:    Rails respond_with and Singular Resources
tags:     rails
---

I ran into a weird issue regarding the [Orcid Rails engine](https://github.com/projecthydra-labs/orcid).

The problem was when a user sucessfully created their profile request for Orcid,
the application redirected them to 'https://localhost:3000/orcid/profile_requests.6'.

The router was converting the `id` of the Profile Request to the `:format` portion
of the resource.

## Assessing the Problem

Below is the controller code.

```rails
class Orcid::ProfileRequestsController
  def create
    return false if redirecting_because_user_has_connected_orcid_profile
    return false if redirecting_because_user_has_existing_profile_request
    assign_attributes(new_profile_request)
    create_profile_request(new_profile_request)
    respond_with(orcid, new_profile_request)
  end
end
```

Nothing suspicious.

And here is the routing code.

```rails
Orcid::Engine.routes.draw do
  scope module: 'orcid' do
    resource :profile_request, only: [:show, :new, :create]
    resources :profile_connections, only: [:new, :create, :index]
  end
end
```

After some sleuthing via a debugger.
And reading of the [Singular Resources section of the Rails Guides](http://guides.rubyonrails.org/routing.html#singular-resources);
And thinking about the generated routes and their helper methods ([see below](#routes-output)).
I realized that the terse version of `respond_with` wasn't going to work.

### Routes Output
```console
$ rake app:routes
Routes for Orcid::Engine:
   profile_request  POST /profile_request(.:format)     orcid/profile_requests#create
new_profile_request GET  /profile_request/new(.:format) orcid/profile_requests#new
                    GET  /profile_request(.:format)     orcid/profile_requests#show
```

## Breaking It Apart

So I needed to tease apart the `respond_with` into two sections.
One for success. And one for failure.

```rails
class Orcid::ProfileRequestsController
  def create
    return false if redirecting_because_user_has_connected_orcid_profile
    return false if redirecting_because_user_has_existing_profile_request
    assign_attributes(new_profile_request)
    create_profile_request(new_profile_request)
    if new_profile_request.valid?
      respond_with(orcid, location: orcid.profile_request_path)
    else
      respond_with(orcid, new_profile_request)
    end
  end
end
```

## Drawing Attention to Surprises

I've been writing Rails apps for awhile, but this was a surprise for me.

So I submitted a [pull request to the Orcid project](https://github.com/projecthydra-labs/orcid/pull/23).
In that pull request I wrote a details on what was going on.
I also crafted a [few tests to state what was happening](https://github.com/projecthydra-labs/orcid/pull/23/files).
