---
layout: post
category: blog
title: Parsing natural(ish) language with Javascript
---

I'm working on a project to convert static html pages into dynamic, interactive study planners for our students.

It's a worthy and large-scale task - students have difficulty understanding the current versions, so we want to build a replacement that takes static data from our CMS, and presents it in an intuitive way.

I'm not about to give a blow-by-blow account of the entire process - I'm writing documentation at the moment, and it's no fun, so I ain't about to repeat myself.

But here's a fun little task - processing strings to booleans.

Background: each planner consists of a list of courses (or course selections or electives and so on). Each course can have one or more requisite values, being pre, anti and co.

If the requisites for a given course are met, it should be available as an option in the plan.

Typically, a requisite will be a simple string, like this:

`ABC123 or DEF456 or GHI789`

That's nice and simple - we can check the previously completed courses, if any of them exist in that string, the requisite has been met.

But of course, it's not always simple. Life would be boring if that were the case. We also have requisites like this:

`(ABC123 or DEF456) and GHI789`

or

`(ABC123 and DEF456) or GHI789`

or

`(ABC123 or DEF456) and (GHI789 or JKL123)`

And so on ad-nauseum. Then there's also cases where the string contains arbitrary text - administrators can add any value they like, and they particularly like to include variations on a theme. Think things like 'a' or 'any' or plural vs singular.

So my first approach no longer works. If it's all `or`, we're sweet, but `and` and `()` make things more complicated.

Given that I ultimately needed a boolean value, I need to distill that string down to something I can evaluate, and hopefully get back the required boolean.

Which didn't turn out to be too tricky at all - funny how a bit of clear thinking can make a daunting task much more manageable.

First up, let's turn those `and` and `or` instances into something useful:

```js
var pre = item.prerequisites.replace(/\s(or)$/gi, '').replace(/and/gi, '&&').replace(/or/gi, '||');
```

The first replace manages instances where a string has a trailing `or`. Yup, it's not the cleanest of data.

Then, I loop through the set of completed courses (which are represented by their unique code). Each instance of a code in the string is then replaced with `true`.

That takes care of any simple requisites that are comprised only of course codes and operators.

To manage all the other garbage that may appear, I'm letting our site editors store override values which will always be replaced with `true`.

By treating these as regular expressions, I can manage plurals and variants relatively simply.

So once we've done that, we can evaluate the string and see what we get back. In case of errors, where the string doesn't evaluate and there are no course or program codes in the string, I'm setting the flag to true, along with an additional flag to display a warning in the UI - it's never going to be possible to capture all scenarios through string parsing, but a decent fallback means this doesn't really matter.

```js
try {
    valid = eval(pre);
} catch (e) {
    if (pre.search(courseRegex) === -1 && pre.search(programRegex) === -1) {
        valid = true;
        item.reqAlert = true;
    } else {
        valid = false;
    }
}
```

Is it foolproof? Absolutely not, someone will always be able to build a more foolish fool. But what it does do is handle 95% of scenarios elegantly, while providing a robust fallback for the outlying cases.
