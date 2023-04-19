Functions support two process models for .NET class library functions:

* In-Process - Azure Functions host is the runtime for .NET functions. Before .NET 5 C# function apps run in the same process as the host. Sharing a process has enabled unique benefits to .NET functions, most notably a set of rich bindings and SDK injections. However, as a side effect, dependencies could conflict (like `Newtonsoft.Json`) and running in the same process also means that the .NET version of the user code must match the .NET version of the host.
* Out Of Process (Isolated) - Out of process is running functions as an isolated process from the main Azure Functions host. This allows Azure Functions to rapidly support new versions of these languages without updating the host, and it allows the host to evolve independently to enable new features and update its dependencies over time.

The two different process models allow for different configurations and features:

* Dependency injection for out-of-process functions is done via the usual .NET core pattern while in-process functions need a custom StartUp class.
* Out-of-process functions support middleware configuration.
* In-process functions support rich binding classes while out-of-process binding classes are simple DTOs, strings and bytes.

.NET 6 functions support both in-process and out-of-process options, but out-of-process does not support all bindings and features supported in in-process. With .NET 7, out-of-process Functions now support a full set of features so out-of-process is the only supported option. This is why we choose the "Isolated" worker when creating a new Functions project. If you want to know more, take a look at the [Guide for running Azure Functions in an isolated worker process](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=browser&pivots=development-environment-vs#differences-with-net-class-library-functions).

### Move from in-process to isolated function

When you want to move from the in-process Azure function to the isolated Azure function, the first thing you need to do is set the `OutputType` property in your `.csproj` file to `Exe`. Then, change your `FUNCTIONS_WORKER_RUNTIME` app setting in `local.settings.json` to `dotnet-isolated`. At this point, your project won't build. Have no fear, you just need to add the startup code.

#### Creating Host

Now you need to write your startup code to make the functions available to the host. In order to do that you will require three packages to be installed:

* Microsoft.Azure.Functions.Worker
* Microsoft.Azure.Functions.Worker.Extensions.Abstractions
* Microsoft.Azure.Functions.Worker.Sdk

Now add the ``Program.cs`` file, delete everything and write (copy and paste) the following code:

```c#
using Microsoft.Extensions.Hosting;

var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults()
    .Build();

await host.RunAsync();
```

#### Dependency Injection

If you use dependency injection into a function app, you just need to add a call to the ``ConfigureServices`` method on your host builder:

```c#
var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults()
    .ConfigureServices(services =>
    {
        services.AddScoped<IService, Service>();
    })
    .Build();

host.Run();
```

At this point, you can go ahead and remove ``Microsoft.NET.Sdk.Functions`` and any package reference to Azure Functions or WebJobs that don't include the term ``Worker``, because they are meant to be used with in-process function apps.

#### Rebuilding Trigger Functions

The first thing you need to do is install the package ``Microsoft.Azure.Functions.Worker.Extensions.Http``, which will give you access to the types required by your HTTP Trigger function. Then, make the following changes:

* Change the request type from `HttpRequest` to `HttpRequestData`
* Change the response type from `IActionResult` to `HttpResponseData`
* Change the method attribute from `FunctionName` to `Function`

This means, for example, your HTTP trigger function should look like this:

```c#
[Function("YourFunction")]
public async Task<HttpResponseData> Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData req)
{
    // your logic...

    return req.CreateResponse(HttpStatusCode.OK);
}
```
