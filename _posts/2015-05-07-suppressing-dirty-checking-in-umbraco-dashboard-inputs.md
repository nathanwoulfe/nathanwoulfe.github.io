---
layout: post
title: Suppressing dirty checking in Umbraco dashboard inputs
category: blog
---

I've been playing with a new Umbraco dashboard package - more on that later - and had run into an issue where Umbraco's inbuilt Angular goodness was dirty-checking inputs in my dashboard.

On it's own, that's a good thing - it's one less task for package developers to worry about. If inputs in the dashboard have their values modified, Umbraco displays an appropriate notification when the user attempts to leave the current view.

That's great, except in my case, I don't care if the inputs are dirty. I'm happy for the user to leave the view whenever they choose - the dashboard I'm building has an explicit save function, but I'm not updating/resetting/clearing the inputs after a successful save (in fact, I'm repopulating them with fresh data, so the inputs are always going to be dirty).

There is, thankfully, a simple fix - decorating the inputs with a no-dirty-check attribute.

Good-guy [Andy Butland](http://twitter.com/andybutland) added the [directive in Umbraco 7.2](https://github.com/umbraco/Umbraco-CMS/blob/7.2.0/src/Umbraco.Web.UI.Client/src/common/directives/validation/nodirtycheck.directive.js), it's just not documented anywhere (at least not anywhere readily discoverable).

Could I have written my own custom directive, yes, but it's great to see useful extensions being added to the Umbraco core.
