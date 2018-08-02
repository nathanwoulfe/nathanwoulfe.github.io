---
layout: post
title: Exposing Umbraco's backoffice on the front-end
category: post
---

Wouldn't it be nice if we could preview Umbraco content without logging in to the backoffice?

We could then share links to yet-to-be published content, essentially allowing offline (as in, non-Umbraco) users to participate in the editing and approval process.

Version 8, and it's rebuilt caching layer, will apparently make this much simpler as content will exist in the cache in multiple states (or something like that, at least).

For now though, we're left with content previews requiring a user log into the backoffice. Unfortunately, the way preview works, that can't be changed.

What we can do though, is make Umbraco think there is a logged-in user. We can fake it. I'll come back to this.

But first, some context.

Chatting to Koben Digital boss (and recently annointed 2018 Umbraco MVP) Peter Gregory after the first Brisbane Umbraco Unicorns Group meetup last month, he raised the idea of a workflow model that allowed participation from outside the backoffice.

As an example - a media manager might want to approve all media releases before they are published, but other than that interaction, has no requirement to ever enter the backoffice.

Not an unreasonable concept by any measure.

Not unreasonable, and potentially solved by allowing previewing outside the backoffice, and shoe-horning that into Plumber for the workflow goodness.

A shout-out is due here, to Dirk at netaddicts for pointing me in the right direction to get started. H5YR.

So. How do we make it work?

To display a preview, Umbraco builds a snippet of XML to represent the node and its ancestors. Essentially a mini cache (remember, umbraco.config is just a big old XML file).

Based on the presence of a couple of cookies (UMB_PREVIEW and UMB_UCONTEXT), we get the preview sent down to the browser.

The UMB_PREVIEW cookie is created in the `PreviewContent` class, which also builds the XML for the preview (`PreviewContent` is old, and uses a few obsolete bits like `Document` and `User`).

The content of the cookie is a GUID, which maps directly to the name of an XML preview file. Nice.

The UMB_UCONTEXT cookie is the backoffice auth cookie - it's generated when a user authenticates and logs in to the backoffice. Cool.

To allow previewing without logging in, we need to set both those cookies. It won't work without both.

Remember I said we could fake it? Let's fake it.

I won't include the whole solution here, just the guts of it with links to the appropriate bits in the Plumber repo.

### Cookie time ###

We need to modify the processing pipeline nice and early, generate our auth cookie with a valid value, then create our preview data, and all should be peachy.

Inside an `OwinMiddleware` derived class, we can do this:

````csharp
// explicitly remove the auth cookie
HttpContext.Current.Request.Cookies.Remove(UmbracoConfig.For.UmbracoSettings().Security.AuthCookieName);

HttpCookie authCookie = CreateAuthCookie(
    user.Name, // this is an IUser type
    JsonConvert.SerializeObject(userData), // userData? where did this come from?
    GlobalSettings.TimeOutInMinutes,
    UmbracoConfig.For.UmbracoSettings().Security.AuthCookieName,
    UmbracoConfig.For.UmbracoSettings().Security.AuthCookieDomain);

// add the new cookie
HttpContext.Current.Request.Cookies.Add(authCookie);

// lie to Umbraco
var identity = new UmbracoBackOfficeIdentity(userData);

var securityHelper = new SecurityHelper(context);
securityHelper.AddUserIdentity(identity);
````

The full file is available here - https://github.com/nathanwoulfe/Plumber/blob/offline-approval/Workflow/App_Start/WorkflowAuthenticationMiddleware.cs. There's some additional OWIN wire-up required, but won't go into that here.

`userData` is a generated `UserData` object, providing a data structure to store the info required by the authentication cookie.

It is a subset of the full `IUser` structure, which I'm creating in a helper function (CreateAuthCookie, similarly, is a helper for building the cookie).

That data must match an existing backoffice user, so while the preview doesn't require a person physically log in, the user is still authenticated against a backoffice user. It's not anonymous, but it is frictionless.

For my solution, I'm grabbing the user ID from the URL, using it to fetch the `IUser`, then using that to build the `UserData` object. The URL is a custom route for serving the preview (more on that later).

Alternatively, we could setup a preview-only user, store that user ID somewhere and grab it in our code. Regardless of approach, we need a real user.

### Routing ###

Now we have a way to set our auth cookie, and we know how to generate the preview XML, just need to wire 'em together.

In an `ApplicationStarted` method, we can define a custom route:

````csharp
RouteTable.Routes.MapUmbracoRoute(
    "OfflinePreviewRoute",
    "workflow-preview/{nodeId}/{userId}/{taskid}/{guid}",
    new
    {
        controller = "OfflinePreview",
        action = "Index",
        nodeId = UrlParameter.Optional,
        userId = UrlParameter.Optional,
        taskId = UrlParameter.Optional,
        guid = UrlParameter.Optional
    },
    new RouteHandler());
````

This example is my Plumber implementation - I use the combination of node ID, user ID, task ID and the workflow instance GUID to provide sufficient obscurity that the odds of someone guessing a preview URL are pretty much non-existant.

In fact (from a StackOverflow answer), if you can generate one billion GUIDs per second, it would still take 36 years to have a 1.95e-03 chance of a collision. The chances of guessing one are pretty low.

The preview is only rendered if the GUID maps to an active workflow on the node with the supplied ID, where the current task matches the provided task ID AND the given user ID is in the group responsible for actioning that task.

Security through obscurity isn't security, but given then that the URL to the view is system-generated and sent to the email address registered for that user, it's robust enough for me to sleep comfortably at night.

In writing this, I've noticed that the cookie path is the site root - setting that instead to the requested node will mean anyone who manages to build a valid URL will still only have authenticated access to that particular route.

Hitting the page with any of those parameters incorrect simply renders the live page, with an appropriate error message.

The `RouteHandler` bit is responsible for looking up the node to render - it's a standard bit of Umbraco tooling, fully documented over here - https://our.umbraco.com/Documentation/Reference/Routing/custom-routes

There's some additional URL validation happening here too, ensuring the requested path has the correct number of segments for a preview request.

### Display it ###

So now we have a pipeline where the auth cookie is set based on the URL and the route handler providing the correct node content, also based on the route.

Finally, we hit the OfflinePreview controller (referenced back in the route mapping).

The full file is available over here - https://github.com/nathanwoulfe/Plumber/blob/offline-approval/Workflow/Controllers/OfflinePreviewController.cs

It's a standard `RenderMvcController` type, where the Index method checks the request is valid (ie the node, task and user IDs and instance GUID) map correctly to a task.

If we pass that check, the controller calls into a service to generate the preview XML and set the UMB_PREVIEW cookie, then finally returns a static HTML file which displays the content in an iframe (similar to the backoffice preview).

The returned HTML is the entry point to an AngularJs application that provides a subset of the full workflow functionality (approve or reject the current task). How that works is a story for another day.

If the check fails, both cookies are cleared and a new cookie set to indicate the request was invalid - that's then used on the front-end to display the error message.

And that, my friends, is how we fake it (or at least a heavily abridged account of how we fake it). The detail is all in the Plumber repo.

### Things still to do ###

 - Refactor the Umbraco bits that rely on obsolete code
 - Tighten up URL parameter validation
 - Write tests (urgh)
