---
layout: post
title: Customising URLs in Umbraco
category: blog
---

By default, Umbraco builds logical URLs based on the backoffice content structure - usually this works fine, but there are cases when it's useful to be a bit clever.

I'm currently working on adding the fantastic [Articulate blogging package](https://our.umbraco.org/projects/starter-kits/articulate/) to a site, and was not entirely satisfied with the URL structure.

Articulate creates URLs along these lines - /blog/archive/post-slug - where /archive/ can be renamed in the backoffice. What I wanted instead was to remove it all together.

Enter Umbraco's `IUrlProvider` and `IContentFinder`.

I won't pretend to have a deep understanding of Umbraco's request pipeline and how these two gems fit into it, but they essentially let us modify URLs however we see fit, and ensure that teh correct content is returned for the requested, modified URL.

My `IUrlProvider` looks like this:

```csharp
public class ArticulateRichTextUrlProvider : IUrlProvider
{
  public string GetUrl(UmbracoContext umbracoContext, int id, Uri current, UrlProviderMode mode)
  {
    var content = umbracoContext.ContentCache.GetById(id);
    if (content != null && content.DocumentTypeAlias == "ArticulateRichText" && content.Parent != null)
    {
      return content.Parent.Parent.Url + content.UrlName;
    }

    return null;
  }

  public IEnumerable<string> GetOtherUrls(UmbracoContext umbracoContext, int id, Uri current)
  {
    return Enumerable.Empty<string>();
  }
}
```  

All that's happening in here is for content nodes representing Articulate blog posts, I'm modifying the returned url to remove the node's immediate parent. Happy days.

In the `IContentFinder`, we need to ensure a request to a modified URL still returns the correct content. All we have to work with is the URL name of the requested node, and a bit of insight into what doctype to expect at a modified URL.

It's as simple as this (although it can be made more complex by caching the returned node to prevent subsequent lookups):

```csharp
  public class ArticulateContentFinder : IContentFinder
  {
    public bool TryFindContent(PublishedContentRequest contentRequest)
    {
      try
      {
        if (contentRequest != null)
        {
          var path = contentRequest.Uri.GetAbsolutePathDecoded();
          var parts = path.Split(new[] { '/' }, System.StringSplitOptions.RemoveEmptyEntries);

          if (parts.IndexOf("blog") != -1)
          {
            IEnumerable<IPublishedContent> blogItems = UmbracoContext.Current.ContentCache.GetByXPath("//ArticulateRichText");
            var blogItem = blogItems.Where(x => x.UrlName == parts.Last()).FirstOrDefault();

            if (blogItem != null)
            {
              contentRequest.PublishedContent = blogItem;
            }
          }
        }
      }
      catch (Exception ex)
      {
        // do something with the exception.
      }

      return contentRequest.PublishedContent != null;
    }
  }
```

Since we know we've modified blog posts, we can concern ourselves only with nodes with 'blog' in the url. From there, it's just a matter of fetching the nodes of the correct type, checking that the UrlName property matches the requested path, and updating the contentRequest if appropriate.

There's no need to worry about setting the template for Articulate nodes, as the package (apparently) does this for us.

Once these two are sorted, all that's left is to register them on startup:

```csharp
public void OnApplicationStarting(UmbracoApplicationBase umbracoApplication, ApplicationContext context)
{
  UrlProviderResolver.Current.InsertTypeBefore<DefaultUrlProvider, ArticulateRichTextUrlProvider>();
  ContentFinderResolver.Current.InsertTypeBefore<ContentFinderByNotFoundHandlers, ArticulateContentFinder>();
}
```

With this all in place, a node path changes from, for example, `/blog/archive/post-slug` to `/blog/post-slug`, the URL resolves correctly, the page renders, and Bob's your mother's brother.
