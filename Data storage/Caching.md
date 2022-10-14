Caching allows you to store objects in memory that require extensive resources to create. An application can increase performance by storing in-memory data that is accessed frequently and requires significant processing time to create. Caching offers powerful features that allow you to customize how items are cached and how long they are cached.

Advantages:

* Easy to use.
* Significantly improves the performance and scalability of an application by reducing the work required to generate content.

Disadvantages:

* Potentially high memory usage (if not limited).
* Can have stale data (we must update cache).
* Possible concurrency issues (if not handled correctly).

You have to be careful with implementing a cache because an inefficient cache can make applications slower. When a request results in a cache miss (requested data does not currently reside in the cache), it increases the end-to-end response time of an application. This is because the application must request data from the cache, and when the cache doesn't contain the requested data, the primary database must be queried as well. There is an additional call that bears no fruit added to the already-existing database call. In these instances, the cache brings no benefit and adds the cache response time as added latency. If there are a high number of cache misses, then a cache can slow down an application.

### In-Memory Cache

[In-memory cache](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/response?view=aspnetcore-5.0) is the easiest way to add caching functionality to your application. It is used when you want to implement a cache in a single process. This means it won't be shared across multiple instances and will possibly cause issues in these kinds of scenarios. It is also [Thread safe](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.caching.memorycache?view=dotnet-plat-ext-5.0#thread-safety) which makes it an even better choice.

```c#
// Setup and usage example

public class HomeController : Controller
{
    private IMemoryCache _memoryCache;

    public HomeController(IMemoryCache memoryCache)
    {
        _memoryCache = memoryCache;
    }
}

public void ConfigureServices(IServiceCollection services)
{
    services.AddMemoryCache();
}
```

### Redis (Remote Dictionary Server) Cache

Redis is an in-memory data store, which can be used as a distributed cache. Redis works by temporarily storing information in a key-value data structure, i.e. dictionary, and it supports a vast variety of data types (strings, lists, sets, JSON...). Redis is single-threaded, which is how every command is guaranteed to be atomic, including the ones that do multiple things. 

To use Redis in your project, install Microsoft.Extensions.Caching.StackExchangeRedis NuGet package. Then register caching services by calling ``AddStackExchangeRedisCache``:

```c#
public static IServiceCollection AddRedisConfiguration(
    this IServiceCollection services,
    IConfiguration configuration)
{
    services.AddStackExchangeRedisCache(options =>
    {
        options.Configuration = configuration.GetConnectionString("RedisCS");
     	options.InstanceName = "SampleInstance";
    });

    return services;
}
```

To use Redis distributed cache, you can use constructor injection:

```c#
public MyService : IMyService
{
    private readonly IDistributedCache _distributedCache;

    public MyService(IDistributedCache distributedCache)
    {
	    _distributedCache = distributedCache;
    }
}
```

Read more about distributed caching [here](https://learn.microsoft.com/en-us/aspnet/core/performance/caching/distributed?view=aspnetcore-6.0).

### When to use in-memory vs when to use distributed cache

In-memory and distributed cache both have their pros and cons, and which one you should use depends on what suits your scenario best.

If you are caching immutable objects, consistency ceases to be an issue. In such a case, an in-memory cache is a better choice as many overheads typically associated with external distributed caches are simply not there. If your application is deployed on multiple nodes, you cache mutable objects and you want your reads to always be consistent rather than eventually consistent, a distributed cache is the way to go.

If you are looking for an always-consistent global cache state in a multi-node deployment, a distributed cache is what you are looking for (at the cost of a performance that you may get from a local in-memory cache).

An in-memory cache is a better option for a small and predictable number of frequently accessed, preferably immutable objects. For a large, unpredictable number of objects, you are better off with a distributed cache.
