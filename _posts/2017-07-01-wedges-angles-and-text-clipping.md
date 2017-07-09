---
layout: post
title: Wedges, angles and text-clipping
category: post
---

Using clip-path and CSS shapes, we can create complex layouts previously only available to print designers.

CSS has always been pretty good at fitting content into boxes.

If we've wanted to do anything more complex layout-wise, life quickly became hacky and loaded with workarounds.

Enter CSS Shapes, part of the CSS3 specification, which simply put, lets us define a path around which our content should flow.

There are a heap of examples out in web-land, explaining how this works and how you can incorporate it in your layouts, so I'm not going into the nuts and bolts here.

What I do want to do is provide a quick example of how CSS Shapes, combined with the clip-path property, can allow us to set text content inside irregular containers.

For example, a trapezoid:

<p data-height="265" data-theme-id="0" data-slug-hash="ZyqJwG" data-default-tab="result" data-user="nathanwoulfe" data-embed-version="2" data-pen-title="ZyqJwG" class="codepen">See the Pen <a href="https://codepen.io/nathanwoulfe/pen/ZyqJwG/">ZyqJwG</a> by Nathan Woulfe (<a href="https://codepen.io/nathanwoulfe">@nathanwoulfe</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

What's going on? I'm using the CSS Shapes spec to define two polygons around which to set the text - these need to be actual DOM elements, so using pseudo ::after and ::before is off the table. It mmeans two empty elements, but I can live with that.

The `shape-outside` rule defines the shape, where the coordinates are simple x,y pairs to describe the shape:

```sass
    // left side
    shape-outside:polygon(5% 0,25% 100%,5% 100%);
    // right side
    shape-outside: polygon(95% 0,95% 100%,75% 100%);
```

That controls how the text flows, but we then also need to clip the element to only keep the part of the element outside the text:

```sass
    // left side 
    clip-path: polygon(0 0,20% 102%,0 102%);
    // right side
    clip-path: polygon(101% -4%,101% 101%,81% 101%);    
```

The polygon definitions are different for the two rules, to add padding between the edge of the shape and the edge of the clip.

Just like anything CSS-related, there's probably a raft of different ways to achieve the same effect.

One benefit of this approach is that the technique could be distilled into a SASS mixin, where the input parameters would be the width of the wedge and the padding distance:

```sass
    @mixin clip-wedge-left($width, $padding) {
      shape-outside: polygon($padding 0, ($padding + $width) 100%, $padding 100%);
      clip-path: polygon(0 0, $width 102%, 0 102%);
    }
```

The width of the wedge is relative to the shape, not the parent container, so in this case, given the `shape-left` and `shape-right` elements are 50% wide, the 20% wedge is actually 10% of the parent container (ie the black trapezoid).

The clip path is slightly larger than the container, just to avoid any unsightly sub-pixel whitespace.

This won't work as a responsive solution without a bit of javascript to calculate the height of the floated elements on resize, but for the sake of this example I've set fixed heights. For unknown text lengths and/or different device widths, throw a little JS at it.

Like most interesting CSS stuff, this one needs to be prefixed, but you're using a build task, so that just magically happens, right?
