---
layout: post
title: Stately - page-state icons in Umbraco 7+
category: projects
---

Long-time Umbraco users would likely be familiar with the Page State Icons package - Stately provides the same functionality for v7 installs.

Based on user-specified config values, Stately appends icons to nodes in the content tree to indicate the value of a particular property on the node.

For example, nodes with a property umbracoRedirect might show a green arrow to indicate the redirect is set.

Settings info is stored in a config file alongside the rest of the property editor files, icons are drawn from the built-in Umbraco icon picker dialog, and I've just added support for popular icon packs, being Belle, Font Awesome and Google Material Design.

How does it work? 

Most of the action takes place in the StatelyTreeEvents class, which inherits from Umbraco's ApplicationEventHandler to expose the tree node rendering events.

In the ApplicationStated method, I'm registering an event - TreeNodesRendering.

{% highlight c# %}
protected override void ApplicationStarted(UmbracoApplicationBase umbracoApplication, ApplicationContext applicationContext)
{
    TreeControllerBase.TreeNodesRendering += new TypedEventHandler<TreeControllerBase, TreeNodesRenderingEventArgs>(this.TreeControllerBase_TreeNodesRendering);
}
{% endhighlight %}

The method that gets called on the rendering even looks like this:

{% highlight c# %}
private void TreeControllerBase_TreeNodesRendering(TreeControllerBase sender, TreeNodesRenderingEventArgs e)
{
    if (!(sender.TreeAlias == "content"))
        return;
    UmbracoHelper umbracoHelper = new UmbracoHelper(UmbracoContext.Current);
    foreach (TreeNode node in (List)e.Nodes)
    {
        int id = Convert.ToInt32(node.Id);
        IPublishedContent _node = (IPublishedContent)null;
        if (id > 0)
            _node = umbracoHelper.TypedContent(id) ?? (IPublishedContent)new ContentService().GetPublishedVersion(id);
        if (_node != null)
            StatelyTreeEvents.AddClassesToNode(node, _node);
    }
}
{% endhighlight  %}

All we're doing is checking first that we're dealing with the content section. If we are, we then iterate each node passed as part of the EventArgs parameter.

We take the node id, check that it is valid, then spin up an IPublishedContent node using that id - we try to grab it from the cache first, if that fails we use the ContentService. Once we have a valid node, we fire off both the TreeNode and the IPublishedContent node to the AddClassesToNodeMethod, where the icon classes are added.

Since a given node can only display a single icon, Stately applies only the first match, so the order of the config elements is important - from first to last, in descending priority.

For each node, Stately iterates the settings object and checks firstly if the current node has the required property. If it does, it's time to do some boolean comparisons.

{% highlight c# %}
var hasValue = PublishedContentExtensions.HasValue(_node, settings.PropertyAlias);
var statelyBool = Convert.ToBoolean(settings.Value);
var propString = PublishedContentExtensions.GetPropertyValue(_node, settings.PropertyAlias);

bool propBool;
bool propCanParse;

if (Boolean.TryParse(propString, out propBool))
    propCanParse = true;
else
    propCanParse = false;  
{% endhighlight %}
    
The first three are obvious - a boolean to represent the presence of a value in the property, a boolean to represent the Stately config value and the current property value as a string.

Next, Stately tries to parse the property string to a boolean - if it passes, the property must be a true/false type, otherwise all we need to know is that it failed.

With those variables in hand, we can then do some comparisons to determine whether or not the current node needs an icon added. For that, there are four possible outcomes which will pass comparison:

- Stately is set to true and the property value parses to true
- Stately is set to true, the property has a value but it does not parse (therefore is not a boolean set to false)
- Stately is set to false and the property value parses to false
- Stately is set to false and the property has no value

From there, it's just a matter of adding the appropriate CSS class to the tree node:

{% highlight c# %}
node.CssClasses.Add("stately-icon " + settings.CssClass);
if (!string.IsNullOrEmpty(settings.CssColor))
{
    node.CssClasses.Add(statelyCSS + settings.CssColor);
}
{% endhighlight %}

The .stately-icon class provides a styling hook for later on, while the conditional colour needs a prefix to avoid Umbraco's native styles adding colours to the menu item text.

The CSS to bring it all together in the rendered tree is pretty straightforward - the biggest hurdle was overriding the default icon styles. These set the icon font on the element decorated with the .icon- class, whereas I need it on the element's ::before pseudo element.

By setting the appropriate font-family based on icon-modifier classes, support for other icon packs was pretty simple - Stately now offers several thousand icon options.

Given this has been a long enough read, I'll break down the settings dashboard in a separate post.

As always, the full source is available on [GitHub](https://github.com/nathanwoulfe/stately), and also as an [Umbraco package](http://our.umbraco.org/projects/backoffice-extensions/stately).

