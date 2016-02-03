---
layout: post
title: Tread carefully, Chrome might start deleting from your filesystem
category: blog
---

Google's Chrome browser is a great piece of kit, when it's not deleting entire folders of work from my file system.

See, I've been fiddling with putting together a Chrome app, just as a lesson in, well, writing a Chrome app.

It's all pretty straightforward and familiar territory for anyone who has ever written a web app - there's just an extra layer of vendor-specific goings-on to take said web app and spit out a Chrome app.

But that's not why we're here.

We're here because Chrome can be very, very naughty.

I loaded my unpackaged app from my local machine, to see what was what.

Gone.

No warning.

No error message.

Nothing. Just an entire folder of development work wiped from my machine.

Not just shunted over to the recycle bin for an easy restore. 

Gone. Up in smoke.

It's a [known bug in Chrome](https://code.google.com/p/chromium/issues/detail?id=378431) - or at least I've been able to find a couple of reports of the same thing happening to other poor souls - but the bug reports don't seem to place any degree of urgency on finding a fix as it's nigh on impossible to replicate. Trust me, I tried - when I have the time, and the inclination, I'll try again.

Thankfully though, I'd been a conscientious developer and had committed my work to a remote git repo.

I didn't have everything in there though, but again, luckily, I'm using Yeoman/Grunt/Bower to manage the project - Chrome obliterated my /app folder, but /dist still had a relatively recent build from which I was able to de-uglify my AngularJs controllers, and get back up and running.

It's a short and pointless story, and could have been a whole lot more painful, if not for source control and dependency management.
