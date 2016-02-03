---
layout: post
title: Indexing Archetype content in Umbraco - Pt II
category: blog
---

While a custom indexer will absolutely get the job done, it's overkill.

There's no real need to spin up a custom indexer for Archetype content, given the exposed events in Umbraco - OnGatheringNodeData in particular.

You might be familiar with this event, and how it can be used to manipulate node data before it is pumped into your Examine indexes. Given that an Archetype fieldset is, at its heart, just a data construct, we can use similar logic to parse out the content we require and add it to our index.

The guts of the code is practically the same as it is when using a custom indexer, only we identify the Archetype fieldset slightly differently:

{% highlight c# %}
private void CustomGatheringNodeDataMethod(object sender, IndexingNodeDataEventArgs nodeData)
{            
    if (nodeData.Fields.ContainsKey("archetypeAlias")) 
    {
        var archetypes = JsonConvert.DeserializeObject(nodeData["archetypeAlias"]);
        var content = "";        
 
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
            if (nodeData.Fields.ContainsKey("bodyText"))
                nodeData["bodyText"] += content;
            else
                nodeData.Fields.Add("bodyText", content);
            }   
        }
    }
}
{% endhighlight %}

That's just very quick cut at how you could index Archetype data - depending on your content configuration the actual process will be different. You might want to index the Archetype data in specific fields rather than just packing it into a generic field like the example.

The custom indexer approach is much better suited to indexing external data sources - given the Archetype content is just a UI wrapped around simple data, the event system is the better way to manage indexing. Either way though, you'll have searchable Archetype data.
