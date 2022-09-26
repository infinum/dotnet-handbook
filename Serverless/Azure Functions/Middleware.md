If you want to add middleware to Azure functions, all you have to do is create a new class that inherits from ``IFunctionsWorkerMiddleware`` and register it in your ``HostBuilder``.

```c#
    public class CustomMiddleware : IFunctionsWorkerMiddleware
    {
        public async Task Invoke(FunctionContext context, FunctionExecutionDelegate next)
        {
            // code before exexecution
            await next(context);
            // code after execution
        }
    }
```

Now register **CustomMiddleware** in the **Program.cs** class where you initialized your host builder. 

```c#
    var host = new HostBuilder()
        .ConfigureFunctionsWorkerDefaults(
            builder =>
            {
                builder.UseMiddleware<ExceptionLoggingMiddleware>();
            })
        .Build();

    host.Run();
```

To learn more, read [here](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#middleware).