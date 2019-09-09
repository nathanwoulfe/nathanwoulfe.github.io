---
layout: post
title: Introducing Plumber - Workflow for Umbraco 8
category: post
---

It's taken a while - longer than I'd hoped - but it's almost time to release a revamped and refreshed version of Plumber, providing end-to-end content workflow in Umbraco 8.

What I'd thought was going to be a quick update and re-release turned into several months of revisioning and refactoring, to make Plumber work with all the new bits in Umbraco 8.

Things like variants and infinite editing.

Things like components and composers.

Things like AngularJs components.

Throw in shifting the CI/CD build from Appveyor to Azure Devops just for fun.

Oh, and dependency injection. And actual tests.

And re-architecting pretty much everything.

And why not add a licensing model to hopefully recoup some of the many, many hours poured into not just P2, but Umbraco 7-friendly P1.

The current release is a beta, so comes with all the usual caveats. Don't use it in production et al. That said, the core workflow engine is much the same as P1, it's everything else that wraps around it that has been modified or updated.

P2 should be migration-friendly for anyone stepping up from Umbraco 7 to 8, but will not run on 8.0.x - minimum version is 8.1.0.

There's still a bit to do, and likely bits that aren't quite doing what I think they're doing, but it's 95% done.

On the licensing-front, I've added a license impersonation feature for testing/evaluation purposes, which will enable all Plumber's functionality on `localhost` only.

I'm using Stripe for subscription management, and while that's not fully configured just yet (the Stripe end is still a test account, the integration is working), feel free to create a test subscription, using Stripe's test card number (4242 4242 4242 4242). Ignore the displayed prices too, those are not final.

To enable license impersonation, download the Pro.lic file from the GitHub release page and drop it into `/app_plugins/plumber`, then restart your site.

All going to plan, you'll have Pro-tier features, all of which should work beautifully (I hope).

Given I'm introducing paid options, the source code is currently private. When I've worked out the optimal way to open source parts, I fully intend doing so.

More than enough fanfare, [here's a link](https://github.com/nathanwoulfe/Plumber-2/releases/tag/v1.0.0-beta), download it, install it, break it, log the issues, make me sad, I'll fix, re-release, etc, so on.
