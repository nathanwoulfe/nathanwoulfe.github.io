---
layout: post
title: p-hold.com - yet another placeholder image service
category: projects
---

There's no shortage of placeholder services offering stock images for designers, yet I thought I'd build another. 

[P-hold.com](http://www.p-hold.com/) serves popular Creative Commons images from [Flickr](https://www.flickr.com/services/developer/api/), via the Flickr API, using PHP to manipulate the images on the fly - think things likes size, colour filters, blur, the typical effects a designer might need when creating layouts. 

A simple caching layer stores recent requests to provide fast responses on subsequent requests for images. 

A cron job hits a class  to polling the Flickr API for the 50 most interesting Creative Commons licensed images.

If it finds a result set, the existing cache and thumbnails are cleared, then we make a couple more trips through the Flickr API - one for the image license details, and another for the photographer's details (the latter is displayed as a watermark on the rendered images, in line with CC requirements).

A thumbnail is stored on the server, and a pair of JSON strings - one with all the photographer details, another with references to each of the images.

Users request images via parameters added to the base p-hold.com URL - 

- /width/height
- /width/height/filter where filter is a hex color code, 'gray' or 'blur'
- /keyword/width/height
- /keyword/width/height/filter
- /u/photographer/width/height where photographer is the Flickr username of the creator
- /ad-size where ad-size is the standard digital advertising element size ie MREC, LREC, LLB etc

The request is rewritten in .htaccess to parse the parameters into a querystring:

{% highlight apache %}
RewriteRule ^([0-9]+)/([0-9]+)/?$ /resize-class.php?w=$1&h=$2&r=$1_$2
{% endhighlight %}

There's a stack of different rules to accommodate the expected URL variants, but all rewrite to /resize-class.php?some-querystring.

Within resize-class.php, [Jarrod Oberto's image resizing code](http://www.jarrodoberto.com/articles/2011/09/image-resizing-made-easy-with-php) does the heavy lifiting. I've extended it to add color filtering/grayscale/blur, which is basic PHP:

{% highlight php %}
if (strlen($c) == 6) {
    list($r,$g,$b) = str_split($c,2);    
    if (imagefilter($im, IMG_FILTER_COLORIZE, hexdec($r), hexdec($g), hexdec($b))) {                            
    $this->placehold($im);
    }
}
{% endhighlight %}

There are some other bits and pieces going on in the resize-image class:

- checking for the image in cache, returning it if it exists
- grabbing an appropriate image from the Flickr API, if the request includes a keyword
- building the attribution watermark
- firing a Google Analytics hit

All the code is on [GitHub](https://github.com/nathanwoulfe/p-hold).

