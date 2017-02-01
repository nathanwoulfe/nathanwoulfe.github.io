---
layout: post
title: Defining custom tree root nodes in Umbraco
category: blog
---

If you've been fiddling with Umbraco for a little while, you may well have stuck a finger into the custom tree or section pie.

Doing so allows us to add our own extensions to the Umbraco backoffice, and define new sections (alongside the default Content, Settings, Media etc), or a new tree (within Users, for example).

This is great. Fun for everyone! But there's an obvious short coming, that you might have already noticed in the default sections - the root node of a tree doesn't display anything. Try clicking the Users node in the Users section - blank.

But, thanks to a bit of Angular magic, we can pump our own views into this blank space, either in our own custom trees or in Umbraco's defaults (which only just occurred to me as I was typing this).

How? Relatively easily.

First, we need to capture the rendering event for the tree root node. We do this in an `ApplicationEventHandler` class, like so:

```csharp
public class UserGroupsTreeEvents : ApplicationEventHandler
{
    protected override void ApplicationStarting(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
    {
        TreeControllerBase.RootNodeRendering += TreeControllerBase_RootNodeRendering;
    }

    void TreeControllerBase_RootNodeRendering(TreeControllerBase sender, TreeNodeRenderingEventArgs e)
    {
        // we're interested in the users node
        if (!(sender.TreeAlias == "users"))            
            return;

        // set the route path to something sensible - this will be displayed in the browser
        e.Node.RoutePath = "/users/dashboard/";
    }
}
```

With that in place, clicking the Users node in the Users section should return a 404 error, since no route exists in the Umbraco AngularJs app.

To solve that, we can just use an AngularJs `$httpInterceptor` to update the request path to our view location. It's a bit messy, and works around the existing Umbraco plumbing, but the result is what we're after.

The interceptor looks like this:

```javascript
var umbraco = angular.module('umbraco');

function interceptor($q) {
    return {
        request: function (request) {
            if (request.url.toLowerCase().indexOf('users/dashboard') !== -1) {
                request.url = '/path/to/the/custom/dashboard.html';
            }
            return request || $q.when(request);
        }
    }
}
angular.module('umbraco').factory('usersDashboardInterceptor', ['$q', interceptor]);

// add the route for the dashboard view
umbraco.config(function ($httpProvider) {
    $httpProvider.interceptors.push('usersDashboardInterceptor');
});
```

All that does is capture requests for the route defined in our event handler, and updates that value to a file we want to serve. The target view is just a normal old AngularJs view and controller combo.
