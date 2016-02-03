---
layout: post
title: Compiling async content (or hey, my directive isn't working)
category: blog
---

The front-end of this site <del>is</del> was previously an AngularJs app, for no other reason than I wanted to build an AngularJs app.

UPDATE: Moving this site to Jekyll threw up a similar issue, where CodePen embeds weren't parsed in the markdown files. Removing the async attribute from the embedded script tag solves this.

Which is great. I have a RESTful API serving up my content (from an Umbraco instance hosted on Azure's free tier), which I can then template and route and manipulate to my little heart's content.

Until I wanted a CodePen embed.

The problem there being that while I can easily add the required Javascript to my template, and send the markup down via the API, the content arrives after - only slightly, but still after - the Javascript has been parsed.

So it didn't work.

I plugged in this [nifty directive by Jurgen Van de Moere](https://github.com/jvandemo/angular-embed-codepen), crossed my fingers and hoped it would magically solve my async woes (while knowing full well that it wouldn't).

It didn't.

Same problem - async content arrives after my template has compiled.

The solution was pretty straightforward - I watch for changes in the object containing the CodePen markup. In the watch, I set a brief timeout, an when that expires, I can traverse the DOM to find the CodePen embed, and recompile it:

{% highlight js %}
Content.getBySlug($routeParams.slug, type).then(function (response)
{
    $scope.html = $sce.trustAsHtml(response.html);
});

$scope.$watch('html', function ()
{
    setTimeout(function ()
    {
        var codepen = document.querySelectorAll('.codepen');
        angular.forEach(codepen, function(elm){
            $compile(elm)($scope);
        });          
    }, 1);
});
{% endhighlight %}

The rest, as they say in the classics, is history. I'm just happy with CodePen embeds.
