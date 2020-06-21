---
layout: post
title: Why I chose to remove paid subscriptions from Plumber
category: post
---

Clickbait! While subscriptions have been removed, there's absolutely still a paid licensing model. 

After a fair bit of thought and plenty of time with the Umbraco 8 version in the wild, I decided the subscriptions experiment wasn't worth it. I'm still hopeful paid licenses will see some uptake, and I feel they offer a better value proposition than the previous subscription model.

Why? Of course I'll tell you why.

For the most part, the free Plumber tier will solve a site's workflow needs. It's a complete workflow engine, with pretty much all the bells and whistles. The number of workflow groups is limited, and some advanced features are disabled, but for many|most sites, it will be plenty powerful.

For those larger sites needing more approval groups, or document-type workflows, or conditional approvals, or non-logged-in approvals, or any of the other advanced features, licenses are available.

The thinking here around swapping subscriptions for one-time payment is thus - if an organisation is running a site at the scale which requires the full suite of Plumber features, it's highly likely that organisation can afford to pay to use Plumber, and that transaction, based on my experience in large organisations, is much easier to complete once, compared to monthly.

It's a whole lot less hassle to approve and reconcile one transaction, than it is to do so monthly (even if the payment is automated, the reconciliation is not).

These are the same organisations that will spend $1000+ a day on contractors, but baulk at $50 a month for Dropbox. Sometimes too, a larger price tag brings a perception of quality (wrongly, of course - Plumber is awesome at any pricepoint :)).

Part of the attraction of adding a subscription model was the potential for ongoing revenue. Rather than a one-off payment, the subscription would generate income indefinitely.

Which all sounds great, except based on the pricing model, it would take three years for one subscription to match the revenue from a single license. After those 36 months, sure, there's extra revenue, but it's a long tail.

It's an exercise in delayed gratification, and I tend to be a little impatient. Plus, from an entirely selfish perspective, that income is a lot more useful now, versus spread over the next three years.

Subscriptions also introduced an additional layer (or several layers) of complexity in the Plumber codebase. I want to be building an awesome workflow tool, not splitting duties between building a less-awesome workflow tool bundled with a subscription management layer.

They meant a dependency on a payment provider (Stripe), and while that dependency didn't ship as part of the Plumber package, it did introduce a dependency on how that provider manages subscriptions. One-time licensing removes that completely - I can change payment providers and nothing need change in Plumber.

I do lose the elegance of being able to create a subscription from within the backoffice. That was nice, it kept users inside Umbraco and made the process friction-free. The changes mean a link out to a payment site, where the license is generated and can then be uploaded via the backoffice.

It's not as slick, but for a one-time process, it's fine. It also means I can use Stripe Checkout rather than a bespoke payment flow, which is so much simpler it isn't even funny.

One upside is that in removing the requirement for generating a trial subscription (and instead creating a trial license internally when no paid license exists on the filesystem), the sign-up flow only happens when paying for a license, it's no longer an onboarding step.

That in turn means I lose some insights into active installs (since trial subscriptions previously created an active subscription in Stripe, I had full records of installs). It's not a huge deal, and between NuGet installs and paid customers tracked by Stripe, I'll still be able to extrapolate something useful.

Removing subscriptions also removes the phone-home code from the Plumber codebase.

Previously, a scheduled task checked the subscription status every 36 hours (also on startup), then would request an updated license file if the subscription period had ended. The remote payment server polled Stripe for up-to-date details, then returned a new license if the subscription was still active.

That's two server hops, two extra failure points, lots more complexity, and little reward. So that's all gone.

No phone-home also means no dependency on Polly, a library for managing faults handling (network connectivity issues, in Plumber's case). No phone-home means no worries about remote sites being available, so no need for Polly. One less moving part.

Unfortunately, such changes mean some things break. Those breakages could have been avoid by leaving obsolete API endpoints running, and maintaining an entire obsolete service layer in both Plumber and on the payment server.

I could have built the updates in such a way that the old UI continued to work. But I didn't. Nothing critical is broken, everything still works as before, but the licensing view in the backoffice will be busted.

That will only affect existing installs, and those installs will all have existing licenses (given that's always been a requirement). When those licenses reach the end of their current subscription period, they won't be able to re-validate (remember, I burned down that API), so will revert to trials, which is fine, because that's all that's currently active (ie, I don't need to manage transitioning Basic or Pro subscriptions, since none exist in the wild).

The licensing section will stay broken, but the license will continue to work. Similarly, upgrading to the latest version with an existing trial license will just work. Better still, the existing trial license file can be deleted, it's no longer required.

I'm considering this a not-quite-graceful degradation. It's incentive to updated to 1.3.1, where everything is shiney and new and not broken. I was never one for making everything perfect in old browsers, instead leaving an incentive to update. This is a similar tack - break it just enought that the best course of action is to get the latest.

That's as easy as a one-line NuGet command: 

`Update-Package Plumber.Workflow`. 

Do it.
