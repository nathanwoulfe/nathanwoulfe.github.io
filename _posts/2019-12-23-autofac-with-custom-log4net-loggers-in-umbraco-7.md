---
layout: post
title: Using Autofac to inject custom Log4Net loggers in Umbraco 7
category: post
---

For the last little while, I've had an eye on USC codebase updates to make the eventual migration to Umbraco 8 as smooth as possible.

A big part of that is indentifying places where we should be changing existing patterns to use depenendency injection, so that the changes required as part of the v8 migration are minimal.

One of the big (if not biggest) benefits of dependency injection is the ability to swap out code for different implementations of a given interface.

In this case, that interface is ILogger, which lives in Umbraco.Core.Logging (in both v7 and v8). 

In v7, you'll typically access the logging context via `LogHelper`, which provides access to the default ILogger implementation, with methods for logging error, warning, info and debug messages.

That's great, until you want more control over where your log messages are written. By default, everything writes to the UmbracoTraceLog.txt file, which live at `/app_data/logs`.

However, by using LogHelper we've taken a hard dependency on the Umbraco log4net implementation, which may not suit our needs.

In the case of the USC website, we run a bunch of after-hours tasks to update data held in Umbraco but sourced from other systems. Part of those tasks is some non-insignificant logging, so that we can readily audit those tasks.

If we wrote everything into the UmbracoTraceLog file, we'd spend a lot of time sifting through the logs chasing particular messages (remember, we're talking v7 here, so no fancy backoffice log viewer).

Instead, we split logs for different task into different files, by using custom Log4Net appenders and loggers.

The setup for these looks something like this:

```xml
<appender name="MyAppender" type="Umbraco.Core.Logging.AsynchronousRollingFileAppender, Umbraco.Core">
  <file value="App_Data\Logs\MyAppenderLog.txt" />
  <lockingModel type="log4net.Appender.FileAppender+MinimalLock" />
  <appendToFile value="true" />
  <immediateFlush value="true" />
  <rollingStyle value="Date" />
  <maximumFileSize value="5MB" />
  <layout type="log4net.Layout.PatternLayout">
    <conversionPattern value="%date [%thread] %-5level %logger - %message%newline" />
  </layout>
</appender>

<logger additivity="false" name="My.Appender.Namespace">
  <level value="INFO" />
  <appender-ref ref="MyAppender" />
</logger>
```

We define an appender with the required behaviour, and a logger to implement that appender. 

The logger is restricted to a particular namespace by using the `name` attribute. Setting `additivity="false"` prevents writing to the UmbracoTraceLog file - if we wanted messages in both, we set that attribute to true.

But how to tell Umbraco to use this logger instead of the default?

Currently, we do the below, where `MyAppenderLog` is the name of the logger:

```csharp
  private static readonly ILog FileLog = LogManager.GetLogger("MyAppenderLog");
```

This isn't great. ILog is part of the Log4Net library, so we're taking an explicit dependency on the logging framework in any class where we want to be using custom logs. That's even worse when it comes time to migrate to v8, since Log4Net has been replaced by Serilog - means a lot of code updates to change something that we can instead fix now.

Rather than explicitly referencing `LogManager`, instead we can leverage our IoC container to request the specific logger.

For the USC website, we use Autofac for inversion of control. Why? Because that's what we chose six years ago when we started building on Umbraco.

When we come to looking at v8, that will likely include a shift to LightInject, unless there's some magical benefits in Autofac which would prevent us swapping across. Cross that bridge later.

So how do we change from our explicity logging dependency to something decoupled and modular?

First, before we even touch anything IoC-related, we need a logging class.

Umbraco's ILogger is implemented in Umbraco.Core.Logging.Logger, which has a public constructor accepting a reference to a Log4Net logging file.

We could resort to newing up a `Logger` in our class, and passing it a file reference, but that wouldn't meet our DI goals.

Instead, let's derive a class from `Logger`:

```csharp
public class CustomLogger : Logger
{
  public CustomLogger(FileInfo log4NetConfigFile) : base(log4NetConfigFile)
  {
    // empty for now, might do stuff in future 
  }
}
```    

All this class does at the moment is pass the `FileInfo` parameter through to the `Logger` constructor. We could sidestep this class altogether, but in the interest of future extensibility, I've kept it.

Since `Logger` implements `ILogger`, we get all those method implementations for free in our sub-class.

Now, back to the Autofac side of the equation.

In our config, we need to tell Autofac about our new logging type:

```csharp
builder.Register(c => new CustomLogger(new FileInfo(IOHelper.MapPath(Constants.MyLoggerFilePath))))
  .Named<ILogger>(Constants.MyLoggerName);
```

I'm using constants rather than magic strings for all the reasons we would use constants rather than magic strings - compile-time checking, self-documentation, defined once and so on.

The `.Named<>()` method allows us to give the logger a name by which to reference it later, which is critical.

We can register multiple loggers using the same syntax, giving each a different path and name. If we'd opted not to sub-class `Logger`, we'd use `c => new Logger())` instead.

Once we've registered our new logger, we need a means of retrieving it:

```csharp
  builder.Register<Func<string, ILogger>>(c =>
  {
    var cc = c.Resolve<IComponentContext>();
    return named => cc.ResolveNamed<ILogger>(named);
  });
```

This is an Autofac delegate factory. We can use it in our solution to resolve objects from the IoC container without actually exposing it to any managed components.

What does that look like?

```csharp
public class MyApiController : UmbracoAuthorizedApiController
{
  private readonly ILogger _logger;

  public ParkingSpacesController(Func<string, ILogger> loggerFactory)
  {
    _logger = loggerFactory(Constants.MyAppenderLog);
    _logger.Info<MyApiController>("log me a message");
  }
}
```

Rather than injecting ILogger, which would return the default implementation and write out to UmbracoTraceLog.txt, we instead pass a reference to the delegate factory, from which we can request the logger of our choice - since we used constants in our IoC config, we can reuse those here.

The logger we get back from the factory implements ILogger, exposes all the expected methods, and writes to our specified file.

When we ultimately update to v8, there's no need to modify a stack of controllers to change the injected logger, we instead update the implementation of that logger, and everything else should continue to work.
