---
layout: post
title: McTaggart - Auto content tagging in Umbraco
category: projects
---

Metadata. Google et al may no longer be fanboying over keywords, but that doesn't mean they're not super useful for defining relationships between your content.

Thoroughly tagged content nodes define taxonomical (is that even a word?) relationships within a site. Obviously, if two pieces of content share tags, we can assume the content itself is related, if even just loosely.

Think lists like related news at the end of a news article. We can compile those lists by grabbing content nodes that share tags (or keywords) with the current page).

Which is all great, provided someone is adding relevant tags to the content.

Sometimes, content authors are diligent and hardworking, and add appropriate tags to their content - terms that serve to describe what the content is about (not what the author THINKS it is about).

The better the tagging, the more robust the relationships between content nodes. We just don't want people to get in the way.

That's the problem domain I've sought to address with McTaggart, a content tagging package for Umbraco.

McTaggart leverages the Open Calais API to retrieve tag sets for Umbraco content. 

It's nothing complicated. You'll need an API key to use the Open Calais API, and the only other configuration option is to set the property aliases to tag - it's possible to send content from more than one property on a single node, but in reality, sending the main page content is probably sufficient.

The package posts the content string off to the API, waits patiently while the magic happens at the other end, then displays the tags for the happy user, who no longer has to think about what they've written.

The benefit lies in the fact that the returned tags are based on machine analysis of the content, not a human perception of the content - there's a big difference between what a person thinks they've written about, and how a machine will analyse the same piece of text.

It's more thorough, more accurate and faster than waiting for a human to tap tap tap what they believe to be a decent set of tags. 

Source code is on [GitHub](https://github.com/nathanwoulfe/McTaggart), the package is available from the [Umbraco package repo](https://our.umbraco.org/projects/backoffice-extensions/mctaggart-auto-tagging-for-umbraco/).

 
