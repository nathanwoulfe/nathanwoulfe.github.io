---
layout: post
title: Pretty URLs in AngularJs apps
category: blog
---

If you've spent any time in AngularJs-land, you'll be familiar with finding the hash character in your URLs.

In the interests of cleanliness, ease of access and (apparently) crawlability, it might be best to get rid of the hash. Thankfully, it's a simple task.

First, you'll need to inject $locationProvider into your app config, and set HTML5 mode to true:

```js
$locationProvider.html5Mode(true);
```

Still with me? I hope so.

That gets rid of the hash in your URLs, but your server is now going to look for content at all your app's routes, so your AngularJs routing is going to fail and you'll drown in a sea of 404s.

You now need to add some server-side magic, to serve up your index.html page to all requests, so that your app routing will work correctly.

I'm serving my content from an Apache environment, so added the below to my .htaccess, and everything works like a charm:

```aconf
RewriteEngine on
#Don't rewrite files or directories
RewriteCond %{REQUEST_FILENAME} -f [OR]
RewriteCond %{REQUEST_FILENAME} -d
RewriteRule ^ - [L]
#Rewrite everything else to index.html to allow html5 state links
RewriteRule ^ index.html [L]
```

There's a suite of examples for different server set ups over at the [Angular-UI repo on Github](https://github.com/angular-ui/ui-router/wiki/Frequently-Asked-Questions#how-to-configure-your-server-to-work-with-html5mode).
