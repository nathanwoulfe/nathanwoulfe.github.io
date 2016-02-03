---
layout: post
title: eadg.be - all the guitars
category: projects
---

To be honest, I purchased the domain before I had any idea what to do with it.

UPDATE (1 Feb 2016): Since I'm a tightass and don't want to pay for hosting, I've shuttered this project. That's going to be mighty disappointing for the three people who visited the site. Need to come up with a new idea for the domain...

EADGBE, for those playing at home, is the standard tuning for a guitar, so finding eadg.be was available was simply too good to pass up.

Until today I've had a super simple site running on the domain, displaying all the video uploads from my YouTube subscriptions. It was as PHP backend which polled the YouTube API every five minutes, then stored the results in a JSON file on the server. Add an AngularJS UI and it's good to go.

But that was boring (and YouTube are deprecating the API methods I was using). As a learning exercise I decided to rewrite the site on top of an Umbraco instance, hosted on Microsoft's Azure cloud platform, and use WebApi to serve up the content to a new-look AngularJS front-end.

Absolutely it's overkill to run this site on a CMS - I could have just written a simple CRUD application, but I wanted to spend some time with the Umbraco API. Given that I'm creating a node for each image, video or article, there's also scope to expand the feature set in the future.

Without going into line-by-line code review, I've ended up writing a few different controllers:

- One to check the Reddit API for new content and parse the response into Umbraco nodes
- One to check the YouTube API for new content and parse the response into Umbraco nodes
- One to check a range of RSS feeds for new entries, and parse these into Umbraco nodes
- One to poll from the front-end to return content to the user

The Reddit and YouTube APIs have consistent data structures, so I've been able to define models for these, but the RSS feeds are a bit more unpredictable.

I've ended up with a selection of helper methods to parse out the content I need for each entry and to clean up the resultant HTML. I probably could have used an existing library for this, but like I said earlier, wanted the project to be a learning tool (biggest lesson learned - parsing RSS is painful).

Site in all it's glory lives at [eadg.be](http://eadg.be).
