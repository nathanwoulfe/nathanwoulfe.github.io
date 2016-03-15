---
title: Watching variables while using AngularJs' ControllerAs syntax
layout: post
category: blog
---

I've jumped on the `ControllerAs` syntax party bus, and find myself writing clearer, more modular code. Which is fantastic.

Doing so makes my code easier to understand and encourages me to put more thought into what I expose to scope, rather than just binding to $scope and clogging up the digest cycle.

As much as I try to avoid using `$watch()` due to its impact on the digest cycle, sometimes it's necessary. Moreso, used sensibly, it's not going to slow up the UI.

Which is a long-winded introduction to what should be a simple example of how `$watch()` works with `ControllerAs`.

Usually, we'd do something like this:

```js
$scope.array = ['this', 'is', 'my', 'array'];

$scope.$watch('array', function (newVal, oldVal) {
    // do something when $scope.array changes
});
```

Which is great - we can observe changes to the variable and act on those. Using `ControllerAs`, this no longer works.

In that case, assuming for example our controller is aliased to `ctrl`, we do this:

```js
var vm = this;
vm.array = ['this', 'is', 'my', 'array'];

$scope.$watch('ctrl.array', function (newVal, oldVal) {
    // do something when vm.array changes
});
```

Why? `ControllerAs` is essentially adding a property to $scope, referencing the controller. That means watching `vm.array` isn't going to work - the controller property is `ctrl`.

Another way around this is to ensure the same alias is used in the view and controller:

```js
// view
<div ng-controller="MyController as vm">
// controller
var vm = this;
```

Which again, is great, except I prefer to consistently use `vm` in the controller to identify bound objects, and a more descriptive name in the view so it stays clear as to which controller is providing the data, which becomes particularly useful with nested or multiple controllers in a single view.
