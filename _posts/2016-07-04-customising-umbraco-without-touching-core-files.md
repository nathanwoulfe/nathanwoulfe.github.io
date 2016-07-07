---
layout: post
category: blog
title: Customising Umbraco without touching core files
---

One of Umbraco's greatest strengths is that it is easily customisable and extensible. 

Partly due to its open source nature, and partly due to the platform following sensible patterns and practices, it's pretty straightforward to Umbraco do things that a vanilla install doesn't do out of the box.

But. There's always a but. If you dive on in and start making changes to Umbraco's core code, you're just setting yourself up for painful upgrades in the future, when you're spending hours merging files that you really should never have touched.

This is more a case with the backoffice than it is with anything front-end - Umbraco's core functionality is distributed as compiled code, but the backoffice exists as a tempting file tree in your IDE.

It's much too easy to dive into the /umbraco/views folder and start chopping and changing files, which simply put, is not the best way forward.

We're days away from deploying an upgrade from 7.1.2 to 7.4.3, after a couple of years on the former version. As part of that process, I've spent some time unravelling our backoffice customisations to ensure the Umbraco install is untouched.

Two years ago, when we first moved to v7, we didn't know any better, nor did Umbraco's backoffice facilitate the pathway we're taking now - the bulk of the backoffice view was delivered to the browser as a single object.

That's changed now, so that the backoffice is much more modular. It's a collection of components and directives, which means we can work some magic with AngularJs' http interceptors.

An interceptor does exactly what it says on the box - it lets us capture http requests and response objects, and most importantly, modify them.

In the case of the Umbraco backoffice, we can capture requests for a particular directive or the response from a particular API request, and customise the response.

As an example, we don't want our editors to set link targets in the link picker. We could hide the input using CSS, but that still leaves the opportunity for particularly clever editors to modify the CSS and unhide the input.

Instead, we can return our own markup instead of the Umbraco version:

```js
(function () {
    // replace the content picker dialog with new markup, removing the _target option
    function linkPicker($q) {
        return {
            request: function (request) {
                if (request.url.toLowerCase().indexOf('linkpicker.html') !== -1) {
                    request.url = '/App_Plugins/Startup/partials/umbLinkPicker.html';
                }
                return request || $q.when(request);
            } 
        }
    }

    angular.module('umbraco').factory('contentLinkPickerInterceptor', ['$q', linkPicker]);
}());
```

Notice in the new URL value, the path to App_Plugins - I've created a startup package to ensure all the required interceptors and other bits and pieces of backoffice code are loaded and ready when the Umbraco instance is started.

It's a basic example - all that's happening is the request URL is replace with a new path, to a different partial view, which looks exactly like the Umbraco default, but removes the directive markup for the _target input. Simple.

The interceptor itself is just a simple AngularJs factory, and needs to be registered against the $httpProvider, which allows us to make modifications to the $http service. We modify the $httpProvider config like so:

```js
(function () {
    'use strict';

    angular.module('umbraco')
        .config(function ($httpProvider) {
            $httpProvider.interceptors.push('contentLinkPickerInterceptor');
        });
}());
```

Both files then need to be included in the startup package.manifest file to ensure Umbraco loads them.

Essentially anything rendered in the backoffice will have a request for the appropriate partial, so can be captured and modified, without impeding future upgrades.
