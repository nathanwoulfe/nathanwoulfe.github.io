---
layout: post
title: Generating color gradiations using SCSS
category: blog
---

Let's use some CSS pre-processing goodness to generate a colour palette.

By combining an SCSS function with a bit of math that I don't really understand, we can create a palette of n colors, with the number of steps equally splitting the color space between our start and end value.

Make sense? Good. The plan is this - from a start color, an end color and a number representing the number of steps, we'll generate the set of colors.

So, first, we'll define our variables and use some built-in SCSS magic to extract the RGB values from our hex color code:

{% highlight sass %}
$start: #1976D2;
$end: #388E3C;

$sR: red($start);
$sG: green($start);
$sB: blue($start);

$eR: red($end);
$eG: green($end);
$eB: blue($end); 
    
$steps: 12;
{% endhighlight %}

Once we've defined our start and end points, we can start iterating.

{% highlight sass %}
@for $i from 1 through $steps {        
    &:nth-child(#{$i}){
        $r: interpolate($sR, $eR, $i, $steps);
        $g: interpolate($sG, $eG, $i, $steps);
        $b: interpolate($sB, $eB, $i, $steps);

        $color: rgb($r, $g, $b);

        // do what ever we want with $color...
    }
}
{% endhighlight %}

To build out the color value for each step, we need to get our math on - to be honest, I'm not sure what the function is actually doing, I found it online. What does matter though is that it returns the values we need for the given step. It's essentially just incrementing the R, G and B values based on the current step, I think.

The function below returns its output back to the loop above, which renders our actual styles.

{% highlight sass %}
@function interpolate($begin, $end, $step, $max) {
    @if ($begin < $end) {
        @return round((($end - $begin) * ($step / $max)) + $begin);
    }               
    @return round((($begin - $end) * (1 - ($step / $max))) + $end);    
}
{% endhighlight %}

The alternative is to hard-code the color values for each step, which is not only painful but completely insane. What happens when you need to change the number of items, or your designer mate decides the base color should be turquoise instead of royal blue? Pain. That's what.

That's the beauty of pre-processors - they turn CSS, which is basically a declarative language, into something closer to an actual programming language, with, y'know, functions and shit.

And finally, in all it's gradiated glory, the CodePen:

<p data-height="268" data-theme-id="0" data-slug-hash="waXXBr" data-default-tab="result" data-user="nathanwoulfe" class='codepen'>See the Pen <a href='http://codepen.io/nathanwoulfe/pen/waXXBr/'>Generating gradiated colors in SCSS</a> by Nathan Woulfe (<a href='http://codepen.io/nathanwoulfe'>@nathanwoulfe</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script src="//assets.codepen.io/assets/embed/ei.js"> </script>
