---
layout: post
title: How good are SASS loops?
category: blog
---

How good? Really good. And they're even better when you use them to generate content.

Take, for example, a calendar/diary view - you'll want to include some sort of time indicator alongside the blocks in the view.

But of course you don't want to hard-code the hours into your template, and equally unappealing is adding those as CSS pseudo elements.

What you can do if you're using SASS or a similar CSS preprocessor, is build them from within a loop.

Say we needed this:

{% highlight sass %}
.week-view .hour-block:nth-of-type(1)::before { content: '7am'; }
.week-view .hour-block:nth-of-type(2)::before { content: '8am'; }
.week-view .hour-block:nth-of-type(3)::before { content: '9am'; }
.week-view .hour-block:nth-of-type(4)::before { content: '10am'; }
{% endhighlight %}

And so on for a 24-period. That's a painful amount of CSS to be writing - particularly when something needs to change, which of course it will.

Instead, we do this:

{% highlight sass %}
.week-view .hour-block {
    @for $i from 1 through 24 {
        &:nth-of-type(#{$i})::before {
            $j: $i + 6;
            $suffix: 'am';
            @if $j > 12 { $suffix: 'pm'; $j: $j - 12 }
            content: '#{$j}#{$suffix}';
        }                      
    }
}
{% endhighlight %}

Sweet, yeah? It's nothing overly complex - the loop renders the selector, then determines the correct suffix (am or pm) and then calculates the hour value before rendering the lot.

Or, we could do something like this (caveat: the final CSS isn't ideal, but was required to work within the constraints of an existing system):

{% highlight sass %}
@mixin depth($depth: 1) {
    $chain: '';

    @for $i from 0 to $depth {
        $chain: $chain + ' + ul div[style*="padding-left"]';
    }
  
    & #{$chain} { @content; }
}

@mixin fromLeft($x) {
    left: #{($x * 20) + 27}px;
}

.umb-tree ul div[style*="padding-left"] {

    &.stately-icon::before {
        left:27px;
    }

    @for $i from 1 through 10 {
        @include depth($i) {
            &.stately-icon::before {
                @include fromLeft($i);
            }
        }
    }
}
{% endhighlight %}

That's part of the SCSS from my [Stately package for Umbraco](https://our.umbraco.org/projects/backoffice-extensions/stately/) - it's positioning the Stately icons with respect the current nesting level of the main content tree.

There's a few things going on here that might be interesting.

The depth mixin is generating a chained selector to target nodes at any menu level between 1 and 10, while the fromLeft mixin is calculating the left positioning of the Stately icon, again with respect to the menu depth.

The first iteration of the loop will generate this:

{% highlight sass %}
.umb-tree ul div[style*="padding-left"] + ul div[style*="padding-left"].stately-icon::before { left: 47px; }
{% endhighlight %}

While the 10th iteration will generate this monstrosity:

{% highlight sass %}
.umb-tree ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"] + ul div[style*="padding-left"].stately-icon::before { left: 227px; }
{% endhighlight %}

I said earlier, I was writing this styling bound by the constraints that exist in the Umbraco back end - I had to find a way of calculating the menu depth without resorting to adding extra markup to the content tree. Using a loop to generate the CSS means that while the output isn't the most elegant, it's simple to manage and update if/when required.
