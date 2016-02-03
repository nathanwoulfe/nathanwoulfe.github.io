---
layout: post
title: When hidden overflows insist on overflowing
category: blog
---

Hiding container overflow is a pretty simple mission - pick the axis and give it a value.

Except, it doesn't always work.

I've been refining mobile styles for a project which uses a variation on the off-canvas navigation pattern. Selecting a card from the main pane of the page opens a sliding draw to display the appropriate content, sending the card panel sliding off the right-hand edge of the screen. It's here that the overflow is hidden, to avoid a horizontal scroll.

It's nothing earth-shattering, but works really nicely for the application.

Until I started playing with it on mobile. Despite having set overflow-x: hidden, I was still getting the horizontal scroll bar, which then made it possible to zoom out and view all the page content - that which should be off-screen and hidden, and the current selected content item. Absolutely not desired behaviour.

Simple fix? Add user-scalable=no to the page metadata. Nope, still overflowing.

Turns out, browsers that parse the meta-viewport tag also ignore overflow when it's set on the body or html element.

Why? There's probably a really good reason. I just don't know it.

It's easy enough to fix, although not entirely semantic - adding a div element to wrap the page content, then setting the overflow on this, works fine. In many cases this container will already exist, so it's just a CSS update. 

 
