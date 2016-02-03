---
layout: post
title: IE11, Windows 8 and fixed background images
category: blog
---

While Internet Explorer 11 is vast improvement on its ancestors, it's not without oddities of its own.

Case and point - fixed background images.

I'd put together a simple campaign website which featured fixed background images on some of the containers. Nothing complicated there, just simple CSS.

Problems though when that page was viewed in IE11 on a Windows 8 machine - the background images stutter and jump when the page is scrolled.

This only happens when the page is scrolled with the mouse wheel, using the scrollbar is fine. The degree of stutter and jump actually varies depending on the mouse being used at the time.

The bug is apparently related to IE11 shipping with Smooth Scroll turned on by default. So while disabling Smooth Scroll will correct the behaviour, that's hardly a viable solution - you can't ask your users to change their browser settings before scrolling your page, can you?

The fix, thankfully is a relatively simple one - moving the background image to the body element.
