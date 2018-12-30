---
layout: post
title:      "My Sinatra project - a primer on bi-directional AR associations"
date:       2018-12-30 11:00:36 +0000
permalink:  my_sinatra_project_-_a_primer_on_bi-directional_ar_associations
---


I organize events in my local area for [Product Hunt](https://www.producthunt.com/)– a very popular community platform for startups to get feedback and exposure - and I wanted to build a startup mentoring database and CMS. This  would give attendees to my events a directory in which to list their startup and find support from mentors who are also listed within the directory platform.

The main functionality is as follows: 

*  A user can signup and create an account 
*  A user can create one or more directory entries for startups they are involved with (known as 'startup profiles')
*  A user can create a mentor profile for him or herself. Only one mentor profile per user account can be created.
*  A user can select any number of startups to mentor
*  A user can only edit and delete the startup database entries and mentor profiles that he/she has created
*  Only users who are logged in can browse Startup and Mentor profile listings 

For the purpose of clarity, let's review some basic definitions:

**Startup profile** - this is a database entry for an existing business consisting of company data
**Mentor profile** -  in order for a user to be a mentor and provide mentorship services to startups, they must submit some basic data about themselves
**User** - a user can create startup profiles and/or have many mentorships. They can just mentor or just be content creators

Some of the associations and relationships between data entities were implicit in the functional  requirements listed above. For instance, it is clear that the User model would have a `has_many` relationship with the StartupProfile model. However, given that a user could submit their startup information and also create a mentor profile (thus being a user and a mentor simultaneously), the task of working out how the data model and the class associations should be set up between the mentor, user and startup was not as straightforward.

Would I need three separate models to adequately capture the relationship? Would the user and mentor have a `has_one` relationship with each other? Or would the mentors have a `has_many` relationship with startups through the users? Perhaps I needed a join table as the user and mentor are the same entity?

After a day of banging my head against many brick walls, I threw in the towel and got some help. My technical support coach came to rescue and suggested I create multiple associations using the same user. In other words, point both user and mentor associations to the user table using some ActiveRecord Magicry.

![](https://i.imgur.com/KF6aZNi.png?2)

To implement the relationships depicted above in the ER diagram, I created two model files – the User model and the StartupProfile model and set the more straightforward `belongs_to` / `has_many` relationships in the respective files. Let’s take a look at what happens inside when we describe these relationships:

![](https://i.imgur.com/zXFPVkW.png?2)

In the Startup Profile model, ActiveRecord derives the class name by taking `:user` and camelizing it to get `User`. If you were to specify a `class_name` option, that would take priority. One of the characteristics of `belongs_to` relationships is that the foreign key (the database column containing the id of the associated user such as `user_id`) is derived by taking the name (user) and adding `_id` . This could be overridden with `:foreign_key`. The first parameter (`:user`) is the name of association, so in particular it is the name of the method you call to get the value of the association. All this means is that we could also have written:

![](https://i.imgur.com/5lJTdiA.png?2)


Clearly this is too verbose, and contrary to best practice so we don’t generally express this association in this way. But the point is, this that you could give the user any name if you didn’t want to use ‘User’ and access the association with the alias, for example:

![](https://i.imgur.com/4teONwT.png?2)

Back to the original problem, given that we have 2 users to associate, a mentor and a user (a user is effectively a startup profile contributor) the associations should use those names. Our foreign keys are `mentor_id` and `user_id`, and in both cases the association will refer to a User, so we need `:class_name => ‘User’`

![](https://i.imgur.com/HKR748f.png?1)

A user is an owner of a startup or a mentor for a startup, and either way it’s a `has_many` relationship. What we are doing here is taking advantage of aliasing and splitting up our `has_many` relationships. However, there is one last essential step to ensure we are completing the association:

![](https://i.imgur.com/nOntASC.png?1)

We need to be explicit in defining the relationship and include the key `inverse_of` because when you  have two models in a `has_many`, `has_one` or `belongs_to` association, the `inverse_of` option tells ActiveRecord that they are two sides of the same association. Without it, we would only be able to create and associate objects in one direction. 

Knowing the other side of the same association Sinatra can optimize object loading so that it will reference the same object in memory, instead of loading another copy of the same record. A model's associations, as far as memory is concerned, are one-way bindings. The `inverse_of` option basically gives us two-way memory bindings when one of the associations is a `:belongs_to`.  With this in place, when a mentor is assigned to a startup profile, the mentorship will now be auto-added to the appropriate user, and vice versa.

It was beyond the scope of this assignment to add an approval workflow for the mentor/startup relationship. In reality, you would never want a ‘mentor user’ to be able to assign themselves at will to a ‘startup user’. In the next iteration of this application, I plan to implement a workflow that includes an email component like Pony to notify the  ‘startup user’ of ‘mentor user’ requests. Only upon approval, would the mentorship association be added. While this seems daunting right now, I am sure there is a way to do it. I invite anyone reading this blog post to collaborate with me on extending this functionality if it seems like an interesting initiative. 

You can find my project [here](https://github.com/schanrai/startup-cms) and my video walkthrough [here](https://drive.google.com/file/d/1LSw33HGOrtf9Q5FMT3XkQ5AvOYTSUWJw/view). 









