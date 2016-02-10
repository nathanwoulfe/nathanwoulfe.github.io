---
layout: post
title: Custom GitHub Pages domains with Cloudflare
category: blog
---

GitHub is a great tool. Cloudflare is also a great tool. Combine the two you get domain name awesomeness.

[GitHub Pages](https://pages.github.com/) lets you host websites directly from a repo - with a couple of clicks you have a site hosted at `username.github.io`.

Which is great. Free hosting is better than not-free hosting, yeah? The Pages doco explains how it all works with regard to user, organisation and project sites.

The next obvious step is to turn that `github.io` domain into something a bit more representative of the site content. 

Making that happen requires a few simple DNS updates which can be done through your registrar - or using Cloudflare.

In my (albeit limited) experience, changes made using Cloudflare propagate considerably faster than those made through my registrar. That may well be a reflection more on my registrar than Cloudflare, but regardless, let us continue.

First step, is create a Cloudflare account. Easy, right?

Then, work through the process of adding a new site. When that's done, you'll be given a pair of nameservers - head over to your registrar and update the existing nameservers with these new ones. Still nice and easy, yeah?

Back at Cloudflare, in the DNS  management tools, you'll likely have a heap of entries - A, CNAME, MX, TXT and so on. 

Add (or edit existing, if it's there) a CNAME record where the `name` is your domain, and the `value` is `username.github.io` (where username is, of course, your user or organisation name).

Add another CNAME record, where WWW points to `username.github.io`.

Now, back in your GitHub repo, you need a CNAME file, the contents of which should be one line - your domain. In my case, that's `nathanw.com.au`.

Cool. Once the DNS changes spread across the interweb, you should be able to hit your GitHub Pages-hosted site via your custom domain.

Any other repos with a `gh-pages` branch will be accessible at `yourdomain.com/repo-name`, which is fine, but we can go one better.

Drop a CNAME file into the `gh-pages` branch, the content of which being the sub-domain at which you want to access the project - I have one hanging off `planck.nathanw.com.au`, so the CNAME file contains exactly that.

Back over at Cloudflare, in the site we set up previously, add another CNAME record. The `name` this time round is the subdomain, pointing again at your GitHub Pages url, the `username.github.io` one (I have, for example, `planck` -> `nathanwoulfe.github.io`).

Once that propagates, your subdomain will be working. Only issue I've seen so far is that the relative path doesn't redirect to the subdomain, but that could always be rectified with a HTTP redirect I suppose.

There's a heap of different tutorials and StackOverflow posts documenting different ways people have set up their own custom domains, but this is the most effective for me - Cloudflare provides a much more robust set of DNS management tools than my registrar, along with all the other useful bits and pieces (see: CDN, analytics, caching etc).

So what have we learned today? You don't need to pay for cheap and nasty hosting for simple, frontend-only projects. Throw Jekyll in the mix and you're sorted.
