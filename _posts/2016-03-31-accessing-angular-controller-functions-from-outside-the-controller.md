---
layout: post
category: blog
title: Accessing Angular controller functions from outside the controller
---

TIL: you don't need to be inside an Angular controller to make Angular magic happen.

It's entirely possible to access controller scope from outside the Angular app itself, which can be incredibly useful in cases where you don't have access to said app - for example, we have an integration between our website and CRM system, allowing web forms to punch data into the CRM.

We can create the markup for a form in the CMS, set a few attributes to populate the Angular model, then render it out to the page, which includes the extra nuts and bolts to make everything work, including the form controller itself.

That's all well and good, until the form becomes more complicated. What if the form is embedded in a page that includes a separate controller, from which we want to pluck a value and use in the form? It can be done, and surprisingly easily.

We can access the controller scope by grabbing one of its elements, and simply asking for it:

````js
var elm = angular.element(document.querySelectorAll('.selector')[0]);
var scope = elm.scope();
````

Happy days, we now have access to the controller scope from outside the controller. From there, we can access any scoped values from the controller. It's even more useful in the case outlined above, where we are working outside the app, and have nested controllers - we can access the parent scope and use it to modify the child scope.

Making use of `injector` means we can also access services from outside the app - in the example below, `contentService` has been loaded elsewhere, but we can make it available to the current scope that we've created.

````js
document.addEventListener('DOMContentLoaded', function (e) {
  $('a.form-link').on('click', function(e) {
    var $parent = angular.element(document.querySelectorAll('.parent-selector')[0]).scope(),
      elm = angular.element(document.querySelectorAll('.selector')[0]),
      $scope = elm.scope();

    // need a service reference? Inject it
    var injector = elm.injector();
    var contentService = injector.get('contentService');

    // $scope.formData exists in the controller, but we can access it outside
    $scope.formData.myField = $parent.ctrl.property;
    $scope.formData.anotherField = $parent.ctrl.anotherProperty;

    // the service data is available in the view - use it for a typeahead source, for example
    contentService.get({count:3000}).then(function (resp) {
        $scope.data = resp;
    });
  });
});
````

This gives the ability to build a framework to manage the bulk of the functionality - taking the form fields from a user-generated form, parsing them and handling the form submission - while also being able to extend that framework with custom code as required, without touching the main application, to add additional logic and associated Angular sprinkles.
