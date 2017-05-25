---
layout: post
title: Revisiting old work doesn't have to hurt
category: blog
---

Spend too much time looking backwards and you're likely to walk into a tree.

Not enough, and you'll miss all manner of valuable lessons. 

I've just spent the better part of a fortnight re-writing an AngularJs web app I put together a couple of years ago. 

At the time, AngularJs was, for me, new territory. I understood the broad concepts, was fascinated by the magic of two-way data binding, and could use the framework to build stuff.

So I built stuff - a timetable planner for the USC website. It was functional, did everything it needed to do and didn't generate any complaints, so was marked off as a success and I marched on to the next project.

Until last month, when said tool needed some new features.

In I dive, to start adding a relatively simple new function. What do I discover? A single, gargantuan controller.

Sure, there was a service (singular) and a couple of filters, but the vast majority of the tool was encapsulated in a single controller.

Which I now know to be bad. Very bad. But at the time, not so much.

We're talking 1000+ lines of Javascript. Lots of $scope. A few $watches.

Also on the shit-list was the distinct lack of comments and code notation. I could vaguely remember what function was responsible for what, but as to how it was working -> info in, magic, info out.

So started a few days of wrapping my head back around the code, pulling it apart, directivising (new word) and servicising (another one) where I could, and unwrapping the monster into much more manageable modules.

State is now managed by factories. There's a pub-sub thing going on too. It's actually pretty elegent. Perfect, no, but much, much better.

As I went, I found bad code. And that's a good thing. Not that I'd written some odd, wobbly functions, but that looking at it now I could recognise the issues.

If I was looking at old work and not finding things I could do better, that would mean I hadn't improved my skills in the last two years. And that would be worse than finding DOM manipulation in a controller.

The same thinking should apply in any job, not just development - if reviewing old work doesn't highlight old deficiencies, I'd argue those shortcomings still exist.

There will still be dodgy code. Time constraints and external pressures will do that. Even in their absence, work done now will seem second-rate in the future.

There's nothing wrong with that. The problem would be with not recognising it.
