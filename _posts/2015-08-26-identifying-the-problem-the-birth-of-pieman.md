---
layout: post
categories: blog,projects
title: Identifying the problem - the birth of PieMan
---

Everyone loves data, right?

It's wonderful in that statistics can be made to tell whatever story best suits the current context.

Low time on page? That's great, our users are finding what they need quickly and easily.

Low time on page? That's not good, our users aren't engaging with our content and are hitting the eject button.

To that end, Google Analytics can be both a God-send and an unkempt demon spawn spewed forth from Satan's quivering loins.

At USC, our web authoring model is very much distributed - we have ~150 active authors maintaining pages across the site, pages of varying complexities and visitations.

We could easily enough grant all those authors access to our Google Analytics account either by way of adding each individual or creating a generic account and distributing the credentials.

Neither is overly appealling, as much because of the somewhat sensitive nature of some of the data in the analytics account, and also because provisioning 150 users would be the shittiest task in a stack of shitty tasks.

Ideally, we'd be providing light-weight analytics data in the context of the author's pages. Old Joe doesn't need to know about what's happening on the other 10,000 pages on the site, he's only really interested in the sub-set that he maintains. He wants to know that his content is engaging and driving visitors to his goal pages.

Sure, each of our authors is contributing to the big picture, but for each individual, the analytics data they're interested in is very much a specific sub-set.

So I wrote a shiny new Umbraco package to meet our need - a light-weight Google Analytics integration that displays a simple set of datapoints, in the familiar context of an Umbraco backoffice node, so that old mate Joe can at a glance compare traffic to his pages since he changed the main image and added some new fluffy text.

Which brings me back to the title of this post.

Being able to implement a robust solution is only part of effective development.

You can't deliver a quality solution without a deep understanding of the problem.

We could relatively easily give access to our Google Analytics account, but while that's a solution, it's showing a lack of understanding with regard to the problem.

When our users ask for page stats, they don't mean the hundreds of datapoints Google generates. They're interested in a few key statistics - time on page, unique visitors, total visits, devices, browser version etc. If they really want something detailed, we can generate a report to suit.

Effectively solving the problem - 'we want stats' - means knowing exactly what that means. I've got a pretty deep understanding of the university's business, particularly with regard to the website, so am in a position to tailor a solution that solves the specific problem without having to spend too much time identifying the exact issue.

It's a great position to be in, made even better by the easy extensibility Umbraco brings to the party, and the super-supportive community members who are always willing to share the approaches they've taken to solving their problems.

I'll add a technical run down on the PieMan package, but for now, if you're interested [take a look at the source](https://github.com/nathanwoulfe/PieMan) or grab the package from the [Umbraco repository](https://our.umbraco.org/projects/backoffice-extensions/pieman-analytics/). It's still in beta, but 90% of the time, it works all the time (see, stats are fun!).
