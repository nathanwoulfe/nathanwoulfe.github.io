---
layout: post
title: Using PrismJs syntax highlighting with AngularJs
category: blog
---

Code samples without syntax highlighting is like Mexican food without sour cream.

NOTE: Since moving this blog to Jekyll and its associated static brilliance, this solution is no longer needed. Still works as documented below, but I'm not doing it on this site any longer.

While it's still ok and reasonably satisfying, you know it could be so much better.

PrismJs is a great Javascript library for adding syntax highlighting to web pages - just include the Javascript and CSS files, make sure your markup is semantically correct (of course it is, right?), and PrismJs does the rest.

Things get a little more difficult if you happen to be using AngularJs or similar to render your pages - there's no content in the DOM when Prism goes looking for it, so you're left with bland, sour cream-less code samples.

A quick search of Stack Overflow questions will return a heap of different directives you can use to Angularify Prism, but for wahtever reason, I couldn't get any of them working. It's probably related to my pretty low-level understanding of Angular.

The solution I've used doesn't require writing a specific directive or adding any additional attributes or decoration to my page content - all I've done is set a short timeout (like, 1ms short) inside of which I manually call Prism.highlightElement(elem).

```js
// Content is my service, getBySlug returns the page by name
Content.getBySlug($routeParams.slug, type).then(function (data) {
    $scope.bodyText = $sce.trustAsHtml(data.BodyText);
});
// bodyText has changed, better check for code...
$scope.$watch('bodyText', function (newVal, oldVal) {
    setTimeout(function () {
        var code = document.getElementsByTagName('code');
        angular.forEach(code, function(c) {            
            Prism.highlightElement(c);
        });
    }, 1);
});
```

Angular purists would likely choke on their coffee, but it does the trick for my use-case.
