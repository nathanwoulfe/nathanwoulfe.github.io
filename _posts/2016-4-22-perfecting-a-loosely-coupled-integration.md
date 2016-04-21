---
layout: post
category: blog
title: Perfecting a loosely coupled integration
---
Just because we're pushing data into a third-party system, doesn't mean our user should be force to interact with said system.

This is even more relevant when the experience we (I'm using the royal we, just so everyone feels warm and included) can offer a more pleasant user experience in our own environment.

We also don't want to lose our dear user to another tab, window or website. We want to keep them in our little corner of the internet for as long as possible.

Rather than punt them off to the other system - in this case, it's our CRM - we'd rather just fire off their data. The user doesn't need to know the context, just that they've successfully completed an action and submitted a form.

To make this work smoothly and provide scope to create new forms without having to release new code, we've done the following:

- Created a generic, reusable container for rendering a CRM form. This includes everything that will be reused from one form to another - basically it's everything other than the form inputs themselves.
- Created a simple AngularJS controller to handle form submission and validation
- Created a WebAPI endpoint on our webserver to transform the form submission
- Created an Umbraco document type to allow creation of the form HTML, which can then be rendered via the aforementioned container

Nothing overly complicated in there. But why the local endpoint? Why not just shoot a POST straight into the CRM?

Dear reader, that's because third-party systems often come with third-party vendors. And because we wanted a loosely coupled integration to keep scope for potentially sending the same data to other or additional systems.

The vendor developed a generic endpoint in the CRM system to which we can POST a JSON (using that term loosely, there's some odd and improper formatting expected, but hey, we'll survive) string and have it processed by the CRM. The CRM shouldn't need to know in advance what fields to expect, but it should be able to process familiar data and ignore unfamiliar.

So we take our form submission, transform it, and submit it to the third-party endpoint. From there, we get a response which can be returned to the user.

Perhaps the most useful feature is that by creating form partials for each instance, we can grab the AngularJS controller scope and modify the form object on the client side, on a per-form basis. Means we can fetch other CMS content to drive typeaheads, or present combo options based on values from other nodes.

Essentially, we don't need to create forms in the CRM. We create forms in Umbraco, as HTML, where we can work whatever magic we see fit, using all the data in the CMS and a bit of AngularJS sparkly stuff. We can move quickly, respond rapidly, and ensure our users aren't bouncing around between systems when all they want to do is RSVP to an event.

