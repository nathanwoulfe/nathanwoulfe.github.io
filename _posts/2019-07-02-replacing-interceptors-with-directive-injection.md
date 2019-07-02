---
layout: post
title: Replacing AngularJs interceptors with directive injection
category: post
---

AngularJs interceptors are a relatively painless means of pre-processing a request, or post-processing a response before it hits the server or client. Super useful for error handling, auth and a raft of other things.

One of those other things that I've done previously is to replace the markup returned by a request for a particular view.

Why? Because sometimes we don't want to (or don't have access to) modify the application code responsible for making the request.

Developing Plumber for Umbraco, for example, I needed to swap out the footer buttons with my own HTML, so that I could introduce my own behaviour (ie submitting into the workflow rather than publishing). In v7, that was reasonably straight forward, due to how the backoffice was put together.

I needed to swap out the `umb-footer-content-right` directive with my own, so my interceptor watched for requests to a URL containing `umb-footer-content-right.html`, then changed the requested URL to one that suited my nefarious needs.

That looks like this:

```javascript
(() => {
    function interceptor($q) {
        return {
            request: req => {
                if (req.url.toLowerCase().indexOf('footer-content-right') !== -1) {
                    if (location.hash.indexOf('content') !== -1) {
                        req.url = '../app_plugins/workflow/backoffice/views/partials/workflowEditorFooterContentRight.html';
                    }
                }
                return req || $q.when(req); 
            }
        };
    }

    angular.module('plumber').factory('drawerButtonsInterceptor', ['$q', interceptor]);
})();
```

The interceptor checks the requested URL, checks that it's a request in the content section, then updates the request object to set my required URL. The request then completes, returns the new HTML, backoffice builds normally and my view is compiled and receives the same scope as the original view would have consumed.

It's a little bit flakey in that it's reliant on URLs not changing, but it works just fine.

Until Umbraco 8.

In v8.1.0, the backoffice build has been changed, for the better. Rather than fetching the HTML views from the filesystem, the build process inlines them into the javascript. Instead of directives referencing `templateUrl`, the compiled javascript includes the full template string in the `template` property. 

This improves backoffice performance by substantially reducing the number of HTTP requests required to fetch all the bits used to build out the backoffice. That's a very good thing. 

For a very quick comparison, a relatively vanilla 8.0.2 backoffice made around 40 requests for HTML assets when loading the editing view for a content node. In 8.1, that drops to around 12. 

For more background to the change, check out the [original issue on GitHub](https://github.com/umbraco/Umbraco-CMS/issues/4845).

While that's all speeding up the backoffice beautifully (both on load and subsequent interactions since the views are all loaded) it means that my interception no longer works since the requests never happen.

Like pretty much everything, there's an alternative solution, which works just as well, if not better.

Instead, I can inject my markup, provide it with any scope I have access to, and recompile the HTML so that it becomes an active part of the AngularJs application.

I do this in my content app controller, since it loads at an appropriate stage of the entire backoffice loading process - importantly, when the app controller is created, it has an ancestor content editor, which includes the footer element I want to target.

Purists will likely argue that I shouldn't be modifying the DOM from within a controller. They are correct, but I'm doing it anyway.

My controller looks like this (truncated for brevity):

```javascript
function appController($compile, $element) {
  const contentForm = angular.element($element.closest('[name="contentForm"]'));
  const footerRight = angular.element(contentForm.find('.umnb-editor-footer-content__right-side'));
  
  footerRight.html('<workflow-hook></workflow-hook>');
  $compile(footerRight.contents())(contentForm.scope());
}
```

When the controller instance initializes, I get the closest element named `contentForm` - this will be the ancestor of the current content app, which is important when we get into infinite editing. We need an element whose DOM location is known and consistent, to enable the traversal in `.closest()`.

`$element` is the element to which the controller is attached, in this case the main view for the workflow content app.

Once we have the form, we can find the footer element, and update its HTML to the required directive markup.

`$compile` essentially takes the provided element (in this case everything in `footerRight`) and converts it into a template, attaches the template function which provides the link between the template and `scope`.

Usually, `$compile` would be used in a directive to update the markup and rebind scope, so the scope provided is typically the current scope, so that the directive can continue to interact with the view.

Not today. Today, I'm linking to the `contentForm` element's scope, so that the re-compiled directive has access to same object as the original footer buttons. If I'd injected `$scope` into the app controller and passed that into `$compile`, the newly compiled footer buttons would be bound to content app controller, which, compared to `contentForm`, is pretty dumb - it doesn't know nearly as much about the current backoffice state.

By grabbing the form element's scope, I've achieved the same as climbing up through parent scopes from the content app controller - think something like `$scope.$parent.$parent.$parent...$parent`, which is pretty filthy.

In my newly injected directive, I have access to all the same properties and methods as exposed to the original footer buttons, so the outcome is functionally identical to the interceptor.

While it's not entirely best-practice AngularJs, it's the type of witchcraft that highlights just how flexible javascript can be if we poke and prod the right spots.
