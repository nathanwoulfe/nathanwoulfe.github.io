---
layout: post
category: blog
title: Fun with gradients and border radius
---

I like to think I have a pretty solid grasp on the nuances of CSS - I spend a good chunk of my time writing, reviewing, refactoring and debugging it.

Even still, it's still pretty easy to unearth new properties or techniques to add to the toolbox.

Take the humble `border-radius` attribute. It makes corners curvy, which is great, especially when you need a curvy corner.

I've been working on a campaign page that derives a lot of its look and feel from comic books, so the design brief featured a two-column layout, styled to look like an open book.

Great, I postulate, chuck a border radius on the top inner corners and it should look all booky-like. Except that curve into the gutter of a book isn't linear. Setting `border-radius:5px` and calling it a day isn't going to cut it.

Which brings me to my great discovery for this week - a border radius value can also be elliptical.

Remember how I said I think I have a good understanding of CSS? Border radius is such a trivial technique, yet I'd never had the need to use an elliptical corner, so didn't even know it was an option. Silly me, hey?

Given that, we can create our bookish corners like so:

