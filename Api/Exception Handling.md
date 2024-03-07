Exception handling logic should be universal for the entire application. All the errors that happen in the API should produce the same error response model. That way the consumers of the API will know how to handle all the errors and where to look for solutions.
### Response model
The error model should be simple, containing an error message (or in some cases multiple errors) and an error code if needed:

```csharp
public class ErrorResponse
{
	public int Code { get; set; }
	public string[] Errors { get; set; }
}
```

It’s a good practice to reference this error model in the Swagger documentation using the `[ProducesErrorResponseType]` attribute on controllers. That way the consumers can generate all the models and easily consume the API.

### Error handler

In order to have the handling logic in one place, we could implement a class with a static method that accepts the exception, processes it, and generates a response. That way, we can use that logic wherever we need it (e.g. API middleware, Azure Functions Middleware, ...). Exception processing consists of determining which exception type was thrown, whether it should be logged, and whether its message should be passed to the consumer when generating an error response:

```csharp
public static class ExceptionHandler
{
	public static ErrorResponse Handle(Exception ex)
	{
		ErrorResponse response;

		if (ex is OurCustomException customEx)
		{
			response = new ErrorResponse()
			{
				Errors = customEx.Errors,
				Code = customEx.Code,
			};
		}
		else
		{
			response = new ErrorResponse()
			{
				Errors = new[] { ex.Message },
			};
		}

		return response;
	}
}
```

### API

ASP.NET MVC comes with a handful of ways to implement error handling in the API layer:

- Overriding the `OnException` method on the controller
- `[HandleError]` attribute on controller methods
- Exception filter
- (Custom) Exception Handler middleware

While all of the available ways have their usages, the one we tend to use the most is Exception Handler middleware. Some of the reasons are:

- Exception handling for the entire application is configured in one place.
- The middleware can be reused in multiple projects.
- Placing the middleware at the beginning of the pipeline enables catching all the exceptions that happen in the rest of the application, be it from other middleware, filters, or services.

Exception handler middleware implementation is quite simple - the main point is to wrap the execution of all the following middleware (i.e. the call of the next request delegate) in a try-catch block and to translate the exception to the correct response in the catch block. Generating a response model from the thrown exception and logging the exception is a crucial part of exception handling, but it’s not the middleware’s responsibility to know such things. That’s why it’s better to extract that logic to a separate `ExceptionHandler` class, following the single responsibility principle.  The middleware should just catch the exception, pass it to the `ExceptionHandler`, get back a response model and write it to the `HttpContext`

Having all that in mind, the implementation of the middleware's `Invoke` method should look something like this:

```csharp
public async Task Invoke(HttpContext context)
{
	try
	{
		await _next(context);
	}
	catch (Exception ex)
	{
		var response = ExceptionHandler.Handle(ex);
		await context.Response.WriteAsJsonAsync(response);
		// Or even better yet, determine the status code based on the exception thrown:
		context.Response.StatusCode = (int)HttpErrorCodes.InternalServerError;
}
```
