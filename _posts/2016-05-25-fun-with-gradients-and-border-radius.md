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

<p data-height="265" data-theme-id="0" data-slug-hash="NrKJjO" data-default-tab="result" data-user="nathanwoulfe" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/nathanwoulfe/pen/NrKJjO/">NrKJjO</a> by Nathan Woulfe (<a href="http://codepen.io/nathanwoulfe">@nathanwoulfe</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

Setting a slightly different radius for each page makes things look a little more natural.

What's missing though, is lighting. We need some gradients to add a bit of depth to our book.

Enter linear gradients:

<p data-height="265" data-theme-id="0" data-slug-hash="dXbrVW" data-default-tab="result" data-user="nathanwoulfe" data-embed-version="2" class="codepen">See the Pen <a href="http://codepen.io/nathanwoulfe/pen/dXbrVW/">dXbrVW</a> by Nathan Woulfe (<a href="http://codepen.io/nathanwoulfe">@nathanwoulfe</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>

Things still look a bit flat down the bottom, and a real book would have page-spread on the outer edges, but it's not a bad representation. Using gradients in the background adds a touch of depth, and the elliptical radiuses (radii?) make things look a little more natural.

So there it is. My learnings for this week - boring old border radius can be irregular, which makes it a whole lot more interesting.
