---
author:   jeremyf
category: practices
filename: 2014-05-06-spitballing-a-design.md
layout:   post
tagline:  Modeling data that will change over time
title:    Spitballing a Design
tags:     ruby, rails
---

*What follows is conjecture and speculation.*

The task at hand:

* To provide suggestions for the metadata we are interested in capturing, but not be bound to those suggestions.
* To acknowledge that our metadata understanding will always be changing

Below is an ongoing sketching out of how I believe works could be arbitrarly
configured for Hydramata. This is a work in progress, but is an attempt to
convey that we may need.

```ruby
# The goal is to define the object that provides our suggestions for user input:
#   * Receive input from the user
#   * Validate the user input based on our guidelines
WorkType.define_type_template(:article) do |article|
  article.fieldset(:required) do |required|
    # :at_least_one would be a custom validator in the application
    required.property :created_by, validates: { at_least_one: true }
    required.property :abstract
    required.property :rights
  end
  article.fieldset(:optional) do |optional|
    optional.property :doi
    # We can send a property to a different datastream.
    optional.property :contributor, datastream: :someOtherMetadata
  end
end

# Define what a given property means. In some cases this may mean exposing
# multiple attributes for this property. (i.e.
# `work_type[created_by][first_name]`)
Property.define(:created_by) do |created_by|
  created_by.predicate = 'http://predicate.org/predicates/name'
  created_by.attributes = [:first_name, :last_name]
  created_by.datastream = :descMetadata
  created_by.validates = { presence: true }
end
```

We would in essence introspect on our object.
By calling fieldset.render we are allowing the fieldset to define how it looks.

```erb
# app/views/works/new.html.erb
<%= form_for(@work) do |form| %>
  <% form.object.each_fieldset do |fieldset| %>
    <%= fieldset.render(form: form) %>
  <% end %>
<% end %>
```

Similarly, we are allowing each property to determine how it should render.
In some cases a property is represented by a set of fields (eg first name, last name).
In other cases it is a single field.

```ruby
def fieldset.render(form: form)
  each_property do |property|
    property.render(form: form)
  end
end
```

```ruby
class WorksController
  def new
    @work = Work.new_form(:article, current_user)
  end

  def create
    @work = Work.new_form(:article, current_user)
    @work.attributes = params[:work]
    @work.workflow = :create_work
    if @work.save
      redirect_to @work
    else
      render @work
    end
  end
end

# @work#save method
def save
  return false unless valid?
  generate_noid!
  sequester_attributes!(noid, attributes)
  enqueue(workflow, noid)
end
```

The goal is to define the sequence of tasks that run.
The tasks would implement a common interface.
These need not be defined in the application.
They are the message that the web application sends to the message queue (i.e. Redis/Resque)

```ruby
WorkFlow.define(:create_work) do |deposit|
  first_task :start
  task :start, { success: :notify_create_work_has_begun }
  task :notify_create_work_has_begun, { success: :generate_job_parameters_by_work_type }
  task :generate_job_parameters_by_work_type, { success: :sequester_remote_files }
  task :sequester_remote_files, { success: :run_anti_virus }
  task :run_anti_virus, { success: :generate_characterization_data }
  task :generate_characterization_data, { success: :generate_derivatives }
  task :generate_derivatives, { success: :generate_permissions }
  task :generate_permissions, { success: :deposit_work }
  task :deposit_work, { success: :notify_create_work_is_complete }
  task :notify_create_work_is_complete, { success: :done}
end
```
How are the attributes sent to Fedora?
We persist a hash that would look somewhat like this:

```ruby
{
  noid: 'abc123def',
  attributes: {
    created_by: [{ first_name: 'Jeremy', last_name: 'Friesen'}],
    abstract: "The valuable abstract!",
    rights: "All Rights Reserved"
  }
}
```

On the file store we have attachments associated with the noid.

With the above, we can create our Fedora objects by adding each of the attributes into the correct datastream.
This may mean that our application requires a DatastreamPersister to help the negotation into Fedora.

When we go to display an object, we would follow the following steps:

```ruby
raw_object = get_object_from_fedora(pid)
work = ObjectInterogator.new
raw_object.datastreams.each do |datastream|
  # Each datastream would a corresponding parser.
  # When we parse a datastream it will ultimately assign all of the properties
  # based on the Property definition. Any properties that are not found would
  # be later processable via a special :junk_drawer fieldset
  work.parse(datastream)
end
```

The above method is, at present, speculation.
The goal is to be able to handle displaying an arbitrary object.
It may make sense to have the object interogator be initialized with an explicit set of attribute/properties.

Interestingly, if we encounter a property on two separate datastreams, they will be presented as one.
It follows that if we update that object that those two property values will shift into a singular "correct" datastream.

## Questions

Should WorkType, Property, and Workflow be defined in the database?
This may provide the greatest customization for adopters.
To complete the Workflow would require validation of existing workers.
Though if a worker doesn't exist that isn't a large problem.

Hydramata would provide seed data for getting that started.
