Middleware is software that is assembled into an app pipeline to handle requests and responses. Middlewares are in some ways similar to MVC Filters: they are run in a pipeline, which means that they define what actions must be done before the next middleware, determine whether the next middleware will be executed, and define what must be done after the next middleware finishes.

Middleware can be defined as inline middleware using the `Use` and `Run` methods in app startup, or in a class that contains the `Invoke` or `InvokeAsync` method. We tend to use the latter approach because that enables the reuse of the same code in multiple places and doesn’t clog up the startup with execution logic.

### Common usages

ASP.NET comes with some commonly used middleware out of the box, some of which are:

- Authentication - adds authentication logic, filling the `HttpContext.User` if the request was authenticated successfully
- Authorization - adds authorization support - this must be defined immediately after the Authentication Middleware
- CORS - configures Cross-Origin Resource Sharing
- Static Files - provides support for serving static files

This list is not exhaustive, to find the rest of the built-in middleware check out the [official documentation](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/middleware/?view=aspnetcore-6.0#built-in-middleware).

There are some cases where we need to create our own Middleware. Here are some examples of such cases:

- Error handling - implementing error handling inside a middleware enables us to handle not only exceptions thrown by controller actions but also exceptions thrown by other middleware, as long as they are executed after this Middleware.
- Authorization - in order to avoid passing the Id of the current user from the controller all the way to the service that actually uses it, we can implement a middleware that fills the user data into a scoped service.

### Execution order

Middleware has access to `HttpContext` and anything added by the preceding middleware. That means that the order we define the middleware is crucial to their successful execution. The order of execution is defined by the order that middleware components are added to *Program.cs* (or *Startup.cs* for earlier projects). If, for example, a middleware expects that authorization is already done, then that middleware must be defined after the `app.UseAuthorization()`.

### Dependencies

Middleware, like any other registered service, can expose its dependencies in its constructor. Those dependencies will be resolved from DI when the middleware is constructed. That being said, have in mind that all middleware is created at app startup, effectively making them singletons. That means that all the services that are not registered as singletons cannot be injected using DI in the constructor, but they can be injected into the `InvokeAsync` method by adding them to the method’s signature:

```csharp
public class VeryImportantMiddleware
{
		private readonly RequestDelegate _next;
		private readonly ISomeSingletonService _singletonService;

		public VeryImportantMiddleware(RequestDelegate next, ISomeSingletonService singletonService)
		{
				_next = next;
				_singletonService = singletonService;
		}

		public async Task Invoke(HttpContext context, ISomeScopedService someScopedService)
		{
				_singletonService.DoSomethingImportant();
				someScopedService.DoSomethingElseImportant();
				await _next(context);
				...
		}
}
```
