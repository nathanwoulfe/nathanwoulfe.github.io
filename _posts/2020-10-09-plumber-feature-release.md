---
layout: post
title: New stuff in Plumber, and a patch release to fix the new stuff
category: post
---

While I'm all for announcing releases on Twitter, sometimes 240 characters just isn't enough.

A week or so ago I released v1.4.0 of Plumber, the workflow engine/platform/solution for Umbraco 8 (can we just say Umbraco now? 8 has been in the wild for ages, and honestly, is anyone releasing packages for 7?), with three lovely new features warranting further explanation.

As is so often the case with a feature release, I've followed it up with a patch to fix a few bits that slipped through to the keeper (cricket analogy), and a couple of minor improvements as well.

On top of the main features there's the usual continual improvement where I'm endeavouring to see Plumber sleeker, faster and more reliable with every release (even if I do end up dropping a patch a week later).

So, let's go back a week and have a look at the new stuff in 1.4.0...

### Attachments

By request, it's now possible to include an attachment when initiating a workflow process.

Why is this useful? It makes it easier to track the external reasons for making a change, and allows much deeper explanation than the workflow comments. 

In large organisations website updates are often made in response to feedback or a request from higher-ups. This change provides a mechanism for associating those requests with a workflow process.

It's also a useful way to ensure that when the blame game starts because someone disagrees with a change, there's additional records to show who asked for such a ridiculous change. Accountability is king.

Attachments are added when starting a workflow, using the familiar Umbraco media picker. Attachments can be turned on/off via a global setting, and are always optional when enabled.

### Group inheritance

Out of the box, Plumber uses its own approval groups for managing workflow permissions. This was a conscious decision to not use existing Umbraco groups, because it's not a given that the roles and responsibilities overlap.

Creating a stack of Umbraco user groups in order to use Plumber didn't sit right with me - Plumber tries to maintain a light coupling to Umbraco, so if you chose to uninstall (GASP!), there's nothing left hanging around in the backoffice.

However, there's still value in being able to leverage existing Umbraco groups, when it makes sense to do so.

So as of 1.4.0, it's now possible to inherit users from existing Umbraco groups. This makes it quicker and easier to add Plumber to an existing site, where Umbraco groups have been used to control user responsibilities. 

It's possible to combine both inherited and explicit membership in the same approval group, so choosing one approach doesn't mean the other is off the table.

### Optional unpublish workflows

Previously workflow configuration has been one size fits all (all being publish and unpublish actions). Not any more.

1.4.0 introduces a global setting to turn off workflow for all unpublish actions, as well as allowing granular control at the node or document type level.

For example, I might want unpublish workflows on all items other than those in my events calendar, where I'm happy for editors to unpublish. By disabling unpublish workflows on my event document type, I get exactly that.

All three of these features are aiming to make Plumber a more well-rounded tool, with options and configuration to cover an increasingly wide range of use cases.

I said at the top that 1.4.0 has been followed by 1.4.1, with a few bug fixes and a couple of minor improvements. 

Bug fixes are boring, but the new bits not so.

The Actions component in the workflow content app now includes a preview button, since it makes sense that an approver might want to eyeball the changes rather than relying only on the editor's comments.

Controls to view attachments and to view differences are now co-located in the Change Description component header. Not a big change, but a subtle improvement.

Over on the admin dashboard in the workflow section, the release notes are now rendered as HTML, and the chart loads a lot faster thanks to fetching and transferring considerably less data.

Is that it? I think so. I hope so. I was going to add images but figure you should fire up Umbraco, install Plumber, and see for yourself.
