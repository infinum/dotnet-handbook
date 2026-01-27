Exception handling logic should be universal for the entire application. All the errors that happen in the API should produce the same error response model. That way the consumers of the API will know how to handle all the errors and where to look for solutions.

### Response model

For response model we recommend using the **Problem Details** (RFC 7807) standard. This standard provides a consistent, machine-readable way to communicate error information. A typical Problem Details response includes:
- **type**: A URI identifying the error type.
- **title**: A short, human-readable summary of the problem.
- **status**: The HTTP status code.
- **detail**: A human-readable explanation specific to this occurrence.
- **instance**: A URI reference to the specific occurrence.

ASP.NET Core has built-in support for Problem Details:
- `Microsoft.AspNetCore.Mvc.ProblemDetails`: For general errors.
- `Microsoft.AspNetCore.Mvc.ValidationProblemDetails`: For validation errors.
- `IProblemDetailsService`: For writing responses and customizing problem details.

It’s a good practice to reference the error model in the Swagger documentation using the `[ProducesErrorResponseType]` attribute on controllers. That way the consumers can generate all the models and easily consume the API.

### Error handler

In order to have the handling logic in one place, we could implement a class with a static method that accepts the exception, processes it, and generates a response. That way, we can use that logic wherever we need it (e.g. API middleware, Azure Functions Middleware, ...). Exception processing consists of determining which exception type was thrown, whether it should be logged, and whether its message should be passed to the consumer when generating an error response:

```csharp
public static class ExceptionHandler
{
	public static ErrorResponse Handle(Exception ex)
	{
		if (ex is CustomException customEx)
		{
			return new ErrorResponse(customEx.Code, customEx.Errors);
		}
		else
		{
			return new ErrorResponse(999, [ex.Message]);
		}
	}
}
```

### API

ASP.NET MVC comes with a handful of ways to implement error handling in the API layer:

- Overriding the `OnException` method on the controller
- `[HandleError]` attribute on controller methods
- Exception filter
- (Custom) Exception handeling middleware
- Implementing IExceptionHandler interface

Up until the release of .NET 8 the preferred way to handle exceptions was to use exception handling middleware. In .NET 8 Microsoft introduced a `IExceptionHandler` interface that gives us a callback for handling exceptions and helps us to separate error handling logic in multiple error handlers.

```csharp
public interface IExceptionHandler 
{
    ValueTask<bool> TryHandleAsync(HttpContext httpContext, Exception exception, CancellationToken cancellationToken);
}
```

Every exception handler needs to be registered using `AddExceptionHandler<YourHandler>()` on the `IServiceCollection` during the API startup.
Every time an exception occurs our handlers get called in the same order as they were registered. `TryHandleAsync` method provides information on whether the current handler can handle the exception. If the handler handles a request, it can return `true` to stop processing. If an exception isn't handled by any exception handler, then control falls back to the default behavior and options from the middleware.

In our projects we would implement custom error handling logic and return Problem Details responses like this:

```csharp
public class CustomExceptionHandler(IProblemDetailsService problemDetailsService) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        if (exception is not CustomException customException)
        {
            return false;
        }

        var problemDetails = new ProblemDetails()
        {
            Status = StatusCodes.Status400BadRequest,
            Title = "Custom error occurred",
            Type = "https://datatracker.ietf.org/doc/html/rfc9110#name-400-bad-request",
            Detail = customException.Message,
        };

        var context = new ProblemDetailsContext()
        {
            HttpContext = httpContext,
            ProblemDetails = problemDetails,
            Exception = customException,
        };

        httpContext.Response.StatusCode = StatusCodes.Status400BadRequest;
        await problemDetailsService.WriteAsync(context);
        return true;
    }
}
```

We also need a default error handler in case some other unexpected exception occurs.

```csharp
public class DefaultExceptionHandler(
    ILogger<DefaultExceptionHandler> logger,
    IProblemDetailsService problemDetailsService) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        logger.LogError(exception, "Exception message: {Message}", exception.Message);

        var problemDetails = new Microsoft.AspNetCore.Mvc.ProblemDetails()
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "Unhandled exception",
            Type = "https://datatracker.ietf.org/doc/html/rfc9110#name-500-internal-server-error",
            Detail = exception.Message,
        };

        var context = new ProblemDetailsContext()
        {
            HttpContext = httpContext,
            ProblemDetails = problemDetails,
            Exception = exception,
        };

        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
        await problemDetailsService.WriteAsync(context);
        return true;
    }
}
```

As you can see in both exception handlers, it is possible to use dependency injection in exception handlers in the same way we would use them in our controllers or service classes.

Before running the API, we need to open `Program.cs` and register problem details and handlers in this way:

```csharp
builder.Services
    .AddExceptionHandler<CustomExceptionHandler>()
    .AddExceptionHandler<DefaultExceptionHandler>();

builder.Services.AddProblemDetails();

var app = builder.Build();

app.UseExceptionHandler();
```

Please note that the **order** of registration matters. Handlers will be invoked in the order they are registered.

In addition to enableing problem details we can also customize them if we have standard or shared information that we want to include in our problem detail responses. Here is an example how to do it:

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = context =>
    {
        context.ProblemDetails.Extensions["traceId"] = Activity.Current?.Id ?? context.HttpContext.TraceIdentifier;

        var userId = context.HttpContext.Request.Headers["X-User-Id"].FirstOrDefault();
        context.ProblemDetails.Extensions["userId"] = userId;

        var instance = $"{context.HttpContext.Request.Method} {context.HttpContext.Request.Path}{context.HttpContext.Request.QueryString}";
        context.ProblemDetails.Instance = instance;
    };
});
```

Please note that by default dot net will include `traceId` extension for us, but if we want to customize it or have fallback we can do it as in example above.
