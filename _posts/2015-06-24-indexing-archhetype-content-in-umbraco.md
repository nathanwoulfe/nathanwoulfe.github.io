---
layout: post
title: Indexing Archetype content in Umbraco
category: blog
---

Imulus' Archetype package is great, but what's the point if your users can't search the content?

Due to the nature of Archetype's data storage, Umbraco's built-in search indexers don't know how to handle it. Which isn't great - if you're putting the time and effort into developing flexible, immersive layouts, you want that content to be indexed and searchable, right?

It's relatively straight forward to push content from Archetype fieldsets into your Examine indexes, and have it searchable alongside the rest of your site.

First, you need a custom indexer, it's going to look something like this:

{% highlight c# %}
namespace Your.Namespace {
    class CustomIndexer : UmbracoContentIndexer {
        protected override Dictionary<string, string> GetDataToIndex(XElement node, string type)      
        {
            our indexing magic goes here...
        }
    }
}
{% endhighlight %}

Now it just needs the magic:

{% highlight c# %}
Dictionary<string,string> data = base.GetDataToIndex(node, type);
string content = "";      
            
if (data.ContainsKey("archetypeAlias")) 
{
    var archetypes = JsonConvert.DeserializeObject(data["archetypeAlias"]);
         
    foreach (var archetype in archetypes)
    {
        //Iterate the archetype objects, add the desired values to the content variable
        if (archetype.Alias == "indexAPropertyOfThisFieldset") 
        {
            content += archetype.GetValue("indexThisProperty")
        }
    }
    //Adding the archetype content to the bodytext field to make it searchable
    if (!String.IsNullOrEmpty(content))
    {
        if (data.ContainsKey("bodyText"))
            data["bodyText"] += content;
        else
            data.Add("bodyText", content);
        }   
    }
    return data;
}
{% endhighlight %}

So what's happening? It's really pretty straightforward - we're checking for a key that matches the alias of our Archetype property. Finding that, we deserialise the data and iterate the Archetype fieldsets.

Depending on your setup, you might have nested fieldsets, repeated content, properties that you don't want to index and so on - you'll need to establish what data you actually want to push into your indexes. Once you know what you're looking for, it's simple matter of grabbing the value and adding it to the content variable.

Once we've iterated the Archetype fieldsets, we can smash the content string into the bodyText field (or any other index field), return the whole data object, and boom, indexed Archetypes.

To use the indexer, just reference it in ExamineSettings.config, like so:

{% highlight c# %}
add name="SearchIndexer" type="Your.Namespace.CustomIndexer, Your.Namespace"
{% endhighlight %}
