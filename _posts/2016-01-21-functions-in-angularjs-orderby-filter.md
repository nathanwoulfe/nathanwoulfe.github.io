---
layout: post
title: Functions in AngularJs' orderBy filter
category: blog
---

AngularJs' orderBy filter, not surprisingly, is really good at setting an array's order.

Per the docs, we simply do this:

```markup
<div ng-repeat="item in array | orderBy: expression">
```

Where expression is a function, string or an array of either.

Which is all great. Work beautifully, until you need to sort by a string that includes whitespace characters.

There's a solution for that, albeit a little hacky:

```markup
<div ng-repeat="item in array | orderBy: '\u0022Order by this string\u0022'">
```

Sure, it sorts, but it's a bit messy and not the most maintainable. It gets worse if that whitespace-laden string is the parent key of a nested property.

Without the whitespace, you can simply do this:

```markup
<div ng-repeat="item in array | orderBy: 'parent.child'">
```

Doesn't work with whitespace in either key. Don't fret though, it's easily solved using a function instead of the string expression.

With this view:

```markup
<div ng-repeat="item in array | orderBy: setOrderBy('Order by this string', 'Then by this string')">
```

This function will return the correct predicate to the orderBy filter:

```js
$scope.setSortOrder = function (a, b) {
    return function (item) {
        if (b === undefined) {
            return item[a];
        } else {
             return item[a][b];
        }
    };
};
```

The function expects up to two keys, but can easily be refactored to handle more. Something like the below accepts an array of strings and uses those to look up the corresponding predicate value:

```js
$scope.setSortOrder = function (arr) {
    return function (item) {
        angular.forEach(arr, function(a) {
             item = item[a];
        });
        return item;
    };
};
```
