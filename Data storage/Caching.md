Advantages:

* Easy to use.
* Significantly improves the performance and scalability of an application by reducing the work required to generate content.

Disadvantages:

* Potentially high memory usage (if not limited).
* Can have stale data (we must update cache).
* Possible concurrency issues (if not handled correctly).

### In-Memory Cache

[In-memory cache](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/response?view=aspnetcore-5.0) is the easiest way to add caching functionality to your application. It is used when you want to implement cache in single process. This means it won't be shared across multiple instances and will possibly cause issues in these kind of scenarios. It is also [Thread safe](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.caching.memorycache?view=dotnet-plat-ext-5.0#thread-safety) which makes it an even better choice.

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
