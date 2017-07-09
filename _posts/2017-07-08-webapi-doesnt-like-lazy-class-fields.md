---
layout: post
title: WebApi doesn't like lazy class fields
category: post
---

Something I learned this week - WebApi controllers won't initialize lazy class fields.

I was working on a piece of code similar to the below, where a WebApi endpoint was referencing a lazily initialized class field:

```csharp
  public class MyApiController : ApiController {
    private Lazy<MyType> MyLazy;
    
    public HttpResponseMessage Get() {
      var lazy = MyLazy.Value;
      // do stuff
      
      return ...
    }
  }
```

The problem being MyLazy.Value was alwayas null. MyLazy was never being initialized.

After spending an inordinate amount of time trying to find a way to refactor the controller to ensure MyLazy was being created, I discovered the sad truth - it would never happen.

Why? Because the controller is initialized in response to the WebApi route request rather than by the application itself (or, at least, that's my understanding).

The solve that, we can essentially proxy the request which relies on the lazy field, by adding a helper class:

```csharp
    public class LazyHelper {
      private Lazy<MyType> MyLazy;
      
      public string DoLazyStuff() {
        var lazy = MyLazy.Value;   
        
        return lazy.SomeMethodReturningAString();
      }
    }
```

We can then use some dependency resolution magic to create an instance of the helper, and access the lazy type: 

```csharp
    var lazy = DependencyResolver.Current.GetService<LazyHelper>();
    var result = lazy.DoLazyStuff();
```

While this does mean creating a new class, it keeps the WebApi controller free of class fields, and essentially creates an interface through which to access the lazy type without worrying about whether or not it will actually be initialized.
