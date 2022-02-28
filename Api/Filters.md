## Filters

API Filters are used to perform various actions and checks before and after specific steps in the MVC pipeline. Filters run within the ASP.NET Core action invocation pipeline, sometimes referred to as the filter pipeline. There are multiple [filter types](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-6.0#filter-types) that are executed at different stages in the filter pipeline.

The fact that filters are run within the pipeline means that they are executed in succession - every filter defines what to do before the next one runs and immediately after the next one finishes its execution. The only exception to that rule is when a filter short-circuits the execution by writing something to the Response property of its context. Short-circuiting the pipeline means that all the following filters will not be run, the execution will immediately begin to fall back down the pipeline.

Filters can be applied to various scopes. When implemented as a `FilterAttribute`, they can be placed on a controller method or on a controller, meaning that they will be executed only when the particular controller action or any action on the controller is being called. In case that we need to run a filter for all endpoints, we can implement one of the available filter interfaces, and then register the filter in the startup configuration.

### Action filter example

In order to demonstrate the usage of filters, we will implement a action filter that measures how long does it take to execute a controller action and then logs it. Logging is done using the ILogger service, which is a scoped service. Since filters are executed inside a request scope, we can use DI to get the required instance. The filter instance is kept alive until all the other filters and actions down the pipeline are done, which means that we can keep the stopwatch as a private variable:

```csharp
public class TimerActionFilter : IActionFIlter
{
  Stopwatch _stopwatch = new();
  ILogger<TimerActionFilter> _logger;

  public TimerActionFilter(ILogger<TimerActionFilter> logger)
  {
    _logger = logger;
  }

  // This method is executed BEFORE the next filter/action is called:
  public void OnActionExecuting(ActionExecutingContext context)
  {
    _stopwatch.Start();
  }

  // This method is executed AFTER the next filter/action is done:
  public void OnActionExecuted(ActionExecutedContext context)
  {
    _stopwatch.Stop();
    var actionName = context.ActionDescriptor.DisplayName;
    _logger.LogInformation(
      "{ActionName} took {Elapsed}ms to execute.",
      actionName,
      _stopwatch.ElapsedMilliseconds);
  }
}
```

To use this filter, all we need to do is register the filter in `Program.cs`:

```csharp
builder.Services.AddControllers(config =>
{
  config.Filters.Add(new TimerActionFilter());
});
```
### Common usages

#### Authentication and authorization

Authentication is usually implemented as a middleware, but authorization is more often run as a filter. That enables us to define different roles and user permissions on different endpoints. Usually that is done using the `[Authorize]` filter attribute, whose constructor parameters allow us to define the required user permissions to run a specific controller action or even all the actions on a controller.

#### Caching responses

Caching action results is often done in a filter because if the required result is found in the cache, the filter can just return it while short-circuiting the rest of the pipeline, saving precious resources.

#### Exception Handling

Although ASP.NET MVC has an `IExceptionFilter` interface that can be used to implement exception handling in a filter, we tend not to use it, but rather to implement a middleware for the job. The reasoning behind this is explained in the [Exception Handling](Exception handling.md) section of this handbook.

### Filters vs Middleware

On a first glance, both Filters and Middleware can be used to solve the same problems: they both wrap the execution of filters/middleware that come after them and can short-circuit the execution if needed. The difference between them is not in the way the code is executed, but rather in what order are they run and in their scopes: Filters are a part of the MVC pipeline, so they are scoped entirely to the MVC middleware, while all other middleware operate on the level of ASP.NET and have access only to the `HttpContext`, alongside anything added by the preceding middleware.

What does this mean in practice? We’ll explore that through a few examples.

Let’s say we wanted to enforce a rule that all requests must be done over HTTPS. We might do that using a filter - that would be sufficient to enforce the rule for all controller actions since they are executed after the filter, but what about serving static files? To serve static files a middleware is used that comes before the MVC. That middleware short-circuits the execution of other middleware, which means that the request will never be processed by the filter, so the HTTPS rule won’t be enforced in this case.

Another common usage example is authentication and authorization. Authentication answers the question “Is this user really who he says he is?”, and to answer the question there is no need to know what the user wants or needs, we only need the token that was provided in the request header. That means that authentication doesn’t need anything from the MVC context, so it could (and should!) be implemented as a middleware. Authorization answers the question “Does this user have the permission to do that?” which requires knowledge of who the user is and what the user is actually trying to do. To know that the authorization logic must have access to the result of authentication and the MVC context in order to determine what action is the user trying to perform and what permissions are required to execute the action. That means that authorization must be implemented in a filter.
