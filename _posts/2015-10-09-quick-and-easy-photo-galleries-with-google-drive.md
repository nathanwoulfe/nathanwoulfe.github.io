---
layout: post
title: Quick and easy photo galleries with Google Drive
category: blog
---

While there are thousands of image gallery plugins floating around, sometimes it's just as easy to re-invent the wheel.

I received a request to 'upload a photo gallery', with more than 350 images. One of our designers had exported a web gallery from the bowels of Adobe's Creative Cloud, that while a nice concept, would have meant adding a new CMS node for each image, plus hardcoding image URLs to the 350+ images and their thumbnail versions, and a few other tasks that are no fun when working in CMS-land.

Instead, I spent a couple of hours - from go to whoa - spinning up a simple AngularJs application to do exactly what we needed, is reasonably easy to reuse in future, and importantly, works pretty nicely.

Given that I had the 700 or so images ready to go, I threw them straight into Google Drive - created a folder for the thumbs, and one for the large images.

The plan was simple - a quick SPA to display x images per page, where each thumbnail is clickable to reveal the full-size image. That means a controller, service, and maybe a directive or two.

There's a heap of documented methods for getting data out of Google Drive.

I decided to simply fire my requests to `googleapis.com/drive/v2/files`, with the appropriate query for the thumbnails and full-sized images.

Smushed into an AngularJs service, that looks something like this: 

```js
.factory('photoService',[ '$http',function ($http) {
    return {
        get: function(urlBase, folderId, key) {
            return $http.get(urlBase + 'q=\'' + folderId + '\' in Parents&fields=items(title,id,imageMediaMetadata)&maxResults=500&key=' + key)
                .then(function(result) {
                    return result.data;
                });
            }
        }
    }
]);

// in the controller
photoService.get(urlBase, folderId, apiKey)
    .then(function (resp) {
        $scope.images = resp.items;
    }
});
```

The parameters to the function are largely self-explanatory - `urlBase` is the URL referenced earlier, `folderId` is the Google Drive folder ID and `key` is the API key.

It returns a handy JSON blob containing everything we need to know about the images - except their URL.

For that, we simply construct it - `docs.google.com/uc?id=imageId`

From there, we can do anything we like with the 700 or so images lurking over in Drive.

Image gallery UIs are stupidly common, so no point drilling through how I ended up rendering this one. Useful though (hopefully) to know how to drag images out of Drive.
