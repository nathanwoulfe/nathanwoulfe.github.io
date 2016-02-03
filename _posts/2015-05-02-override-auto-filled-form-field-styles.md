---
layout: post
title: Easily over-ride auto-filled form field styles
category: post
---

Not a fan of Chrome's pasty yellow backgrounds on auto-filled form fields?

It's trivial to override these and include your own much more attractive styles.

You could choose to disable auto filling, but that's introducing a usability issue - it's a familiar, helpful design pattern, so let's keep it that way.

I'm using SASS and AngularJS in my project, hence the syntax and extra selectors, but the gist of it should be obvious.

{% highlight sass %}
$validColor: #78FA89;
$invalidColor: #FA787E;

input, textarea {
    border-bottom:2px solid #eee;
    &.ng-invalid.ng-touched {
        border-bottom-color: $invalidColor;
        &:-webkit-autofill {
            -webkit-box-shadow: 0 0 0px 1000px $invalidColor inset;
        }
    }
    &.ng-valid.ng-touched {
        border-bottom-color: $validColor;
        &:-webkit-autofill {
            -webkit-box-shadow: 0 0 0px 1000px $validColor inset;
        }
    }
}
{% endhighlight %}

Chrome won't let us override the background color, so the box-shadow essentially overlays our color over it.

I'm adding the AngularJS validation classes to display a green background on valid fields and a red on invalid, after the user has interacted with them - a blank or invalid field triggers the .ng-invalid.ng-touched classes.

Complex? Far from it. Useful? Definitely.
