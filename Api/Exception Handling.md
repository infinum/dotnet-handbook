Exception handling logic should be universal for the entire application. All the errors that happen in the API should produce the same error response model. That way the consumers of the API will know how to handle all the errors and where to look for solutions.
### Response model
The error model should be simple, containing an error message (or in some cases multiple errors) and an error code if needed:

```csharp
public record ErrorResponse(int Code, string[] Errors);
```

It’s a good practice to reference this error model in the Swagger documentation using the `[ProducesErrorResponseType]` attribute on controllers. That way the consumers can generate all the models and easily consume the API.

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

In our projects we would implement custom error handling logic and return appropriate error responses like this:

```csharp
public class CustomExceptionHandler : IExceptionHandler
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

        httpContext.Response.StatusCode = StatusCodes.Status400BadRequest;
        var errorResponse = new ErrorResponse(
            Code: 123,
            Errors: [customException.Message]);
        await httpContext.Response.WriteAsJsonAsync(errorResponse, cancellationToken);

        return true;
    }
}
```
We also need a default error handler in case some other unexpected exception occurs.

```csharp
public class DefaultExceptionHandler(ILogger<DefaultExceptionHandler> logger) : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger = logger;

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(
            exception, "Exception message: {Message}", exception.Message);

        var errorResponse = new ErrorResponse(
            Code: 999,
            Errors: [exception.Message]);
        httpContext.Response.StatusCode = StatusCodes.Status500InternalServerError;
        await httpContext.Response.WriteAsJsonAsync(errorResponse, cancellationToken);

        return true;
    }
}
```

Before running the API, we need to open `Program.cs` and register them in this order.

```csharp
builder.Services
    .AddExceptionHandler<CustomExceptionHandler>()
    .AddExceptionHandler<DefaultExceptionHandler>();
```

As you can see in the `DefaultExceptionHandler`, it is possible to use dependency injection in exception handlers in the same way we would use them in our controllers or service classes.