---
layout: post
title: Versioning, targeting and other such adventures
category: post
---

In case you missed it, Umbraco's .NET Core release is just around the corner.

It's the result of a huge amount of work, stemming from the seed a project birthed at the 2019 Retreat. It will deliver all the benefits of .NET Core (cross-platform Umbraco, anyone?) in your favorite CMS.

It also means a new approach for package developers, as Umbraco 9 will no longer be able to support the tried and true zip package format, instead requiring NuGet installs for all packages.

That's a .NET Core restriction/requirement, and is explored more fully [over here](https://github.com/umbraco/Umbraco-CMS/discussions/10411) and [here](https://umbraco.com/blog/packages-in-umbraco-9-via-nuget/).

For many developers, that means a new approach to how they build and distribute packages. Ultimately though, it means a unified delivery platform, with no more fragmentation between the repository on our.umbraco.com, and what's available via NuGet.

For me, I've been delivering Plumber solely by NuGet since V8 launched, so there's no extra work required in that space. Humblebrag.

The work however was hiding in how to best support V8 and .NET Framework, alongside V9 and .NET Core.

My initial approach was to cut a V9 branch and work through the required code updates to get Plumber up and running on the shiny new Umbraco.

That would in turn require a new build process since the one I've been using is ancient, and .NET Core is not ancient. Sometimes mixing old and new works, this is not one of those times.

So it's a new build process, and a new CI/CD pipeline, to push a new version of Plumber just for V9.

Sounds fine, but it quickly became obvious that this wasn't the ideal way forward (quickly in the sense that I'd done the full migration and had an early alpha ready to go).

The biggest issue with this approach is supporting both the V8 and V9 package versions.

Git makes it easy enough to move change between branches (hello cherry-picking), but that quickly becomes complicated when changes flow in both directions. 

Add a feature in V8, remember to move it to 9.

Make a change in V9, remember to move it to V8, but only if that change is compatible. Maybe it's using new APIs not available in V8, maybe it's completely new code that isn't required at all in V8.

It wasn't a good time. Keeping both in sync through one patch release was painful enough, so back to the drawing board.

The alternative was a single branch generating a multi-targeted build. One common codebase, with conditional code blocks where required, and leaning on build tools to correctly package the right files for the right target framework.

# Make it play nice

For cases where implementation changes between V8 and V9, most can be managed through aliasing or creating implementing new interfaces in the V8 code.

If you've ever pondered the real-world application for interfaces in your code, here's a great example.

In V9, Umbraco drops Serilog in favour of Microsoft's built-in logging API. It serves the same purpose - writing message when things go bad (or good) - but does so via a different interface so code written for V8 won't 'just work' in V9.

To fix that, we create an interface with the required methods (matching the naming of the V9 logger methods), and using conditional code blocks can inject the correct interface depending on whether the current build is Framework or Core.

The interface and its implementation look something like this (albeit with more methods):

```csharp
public interface ILogger<T> {
  void LogDebug(string message, params object[] propertyValues);
}

public class Logger<TEntity> : ILogger<TEntity> {
  private void ILogger _ logger;
  
  public Logger(ILogger logger) => _logger = logger;
  
  public void LogDebug(string message, params object[] propertyValues) => _logger.Debug(typeof(TEntity), message, values);
}
```

The injected logger is from the `Umbraco.Core.Logging` namespace, which only exists in V8. `LogDebug` is the same method signature as in the V9 logger, from the `Microsoft.Extensions.Logging` namespace. Since the latter is generic, our logger must be too.

Our logger needs to be registered via a composer, in code running only in Framework builds.

To use the new logger, we introduce conditional blocks to our class:

```csharp
#if NET472
using Plumber.Core.Logging;
#else
using Microsoft.Extensions.Logging;
#endif
```

This way, we can inject our logger once, using the common interface, and the compiler manages the rest:

```csharp
private ILogger<MyType> _logger;

public MyClassConstructor(ILogger<MyType> logger) => _logger = logger;

public void Log() => _logger.LogDebug("Interfaces can be {Adjective}", "useful"); 
```

We can create our own V8 implementations of V9 interfaces, with the compiler finding the correct dependencies and using our abstractions where appropriate.

This is one of the big benefits of fully embracing dependency injection, as we can defer a lot of the heavy lifting to .NET, and focus on writing streamlined, modular code.

While the multi-targeted approach can mean more code, and potentially more complex code when conditional blocks exist outside of using statements, it does mean that there's no need to jump between branches to view how the same logic works in V8 vs V9. 

It's all in the one place, and with the right amount of abstraction, conditional blocks can be minimal.

# Ugh. Naming.

That's all lovely and I'm back to one branch and one NuGet package to support both V8 and V9, but it leaves the biggest issue - naming.

I kinda stuffed up when Umbraco 8 launched - the original Umbraco 7 Plumber still exists, with its own versioning. For Umbraco 8, Plumber became Plumber2, starting afresh at v1.

Luckily the NuGet package was renamed for Umbraco 8, so there's a glimmer of hope.

Confused? Me too. Sorry.

With Umbraco 9 around the corner, it's ridiculous to have Plumber 1.x for Umbraco 7, Plumber2 1.x for Umbraco 8 and Plumber2 2.x for Umbraco 9, especially given the whole yarn above where I wrote about the joys of a common codebase.

The plan, at the moment, is something like this:

 - The V7 version (Workflow.Umbraco on NuGet) will remain as is. It's not in active development, and will only see critical security fixes. The repository will be renamed to Plumber OG or Plumber Original Recipe or something equally foolish.

 - The V8 and V9 versions (Plumber.Worklow) will no longer be Plumber2. It's just Plumber. Dropping the 2 means Plumber v2.0.0 makes sense, and will be the first multi-target release. v2.0.0 may release with Framework support before the Core version is complete.

 - The current V8 Plumber2 1.x branch (see, ridiculous) will cease development with the v1.6.3 patch release, which was released earlier today (convenient, yah?).

It's onwards and upwards, with a single codebase, easier maintenance and improved naming, which of course is the most important part.



