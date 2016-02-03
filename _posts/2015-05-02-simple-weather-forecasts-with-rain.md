---
layout: post
title: Simple weather forecasts with Rain?
category: projects
---

Unless you're a meteorologist, most weather forecasts are far too detailed.

For me, at least, all I want to know is if it's going to rain.

To play with that idea, learn a bit more about how AngularJS works, and to learn how to merge Javascript objects from different sources.

Basic gist of this project is:

- Ask user for access to their location
- Fire that data off to the Open Weather Map API, asking for data from five stations
- Grab the forecast data from each station
- Mash it together to get one object
- Take the average of the forecasts
- Display it

Code-wise, there's not much to it (it's all on [GitHub](https://github.com/nathanwoulfe/rain)), but the object manipulation is kind of interesting - let's have a closer look, shall we?

The merge function accepts two parameters - being the objects we want to merge. Parameter a is the merged object, b is the object for merging. We unpack a into a cache variable, then unpack b into the same variable.

{% highlight js %}
function merge (a, b) {
    var cache = {};
    cache = unpackObject(a, cache);
    cache = unpackObject(b, cache);
    return cache;
}
{% endhighlight %}

The unpackObject function then iterates the new object. It checks for a value in each property, if one exists it then checks the cache variable for the existence of that property. If it doesn't exist, we add a new property to the cache variable, with the value from the unpacked object.

If the property does exist, and is the same type in both objects, we either send it back through the merge function (if it's an object), or we add the value to the existing property in the cache variable.

{% highlight js %}
function unpackObject (a, cache) {
    for (prop in a) {
        if (a.hasOwnProperty(prop)) {
            if (cache[prop] === undefined) {
                cache[prop] = a[prop];
            } 
            else {
                if (typeof cache[prop] === typeof a[prop]) {
                    if (is_object(a[prop])) {
                        cache[prop] = merge(cache[prop], a[prop]);
                    }
                    else {
                        cache[prop] += a[prop];
                    }
                }
            }
        }
    }
    return cache;
}
{% endhighlight %}

And the outcome? In this case it's an array of weather forecast objects that previous existed as nested objects in the API response, which can then be iterated, averaged, and rendered.

Functioning page is at [nathanwoulfe.github.io/rain](nathanwoulfe.github.io/rain)
