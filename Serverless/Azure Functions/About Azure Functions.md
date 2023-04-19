[Azure Functions](https://learn.microsoft.com/en-us/azure/azure-functions/functions-overview) is Microsoft's serverless solution. They are, unsurprisingly, hosted on Azure, Microsoft's cloud service. Functions support several different programming languages such as C# and JavaScript, F#, Java, Python, and PowerShell.

Azure Functions are interlocked with Azure Storage services, meaning that every Azure function needs Azure storage configured to be able to run. The storage account connection is used by the Functions host for operations such as managing triggers and logging function executions. It's also used when dynamically scaling function apps.

### Scaling

Our scaling options will depend on the hosting plan we have for our service. There are three basic hosting plans available for Azure Functions:

* Consumption plan - Functions scale automatically and billing is calculated only for the used resources. This means that the functions have a cold start (you can read more about its implications in our [Serverless computing chapter](../serverless-computing)). A single-function app in this plan scales out to a maximum of 200 instances, while a single instance can process more than one message/request at a time, except for time-triggered functions which are always singletons.
* Premium plan - Automatically scale using pre-warmed workers which run applications with no delay after being idle. Functions run on more powerful instances and connect to virtual networks. No cold start.
* Dedicated (App Service) plan. - Run functions within an App Service plan at regular App Service plan rates. This plan is suitable for long-running scenarios such as Durable Functions, or where predictable costs are needed.

Regardless of our scaling options, sometimes our requirements might lead us in the other direction. For example, if we needed queue messages to be processed by a function one at a time then we need to configure the function app to scale out to a single instance. In that case, the function's concurrency should be limited too. That is done by setting `batchSize` to 1 in the `host.json`. Learn more [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#concurrency) and [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-concurrency).

### Best practices

* A Function should be **short-lived**, so whenever possible, refactor large functions into smaller function sets that work together and return responses fast.
* Functions should be **stateless** and **idempotent** if possible. Associate any required state information with your data. A function could process data based on that state while the function itself remains stateless.
* A Function could encounter an exception at any time. Design your functions with the ability to continue from a previous fail point during the next execution.
* Plan for scaling, it is important to understand how your function app responds to load and how the triggers can be configured to handle incoming events.
* Newly built functions should be **out-of-process** (you can read more about this in our [In-process and isolated Functions chapter](./In-process-and-isolated-functions)).

### Project creation

Add a new project through the VS Function App template. When choosing the Functions worker, be sure to pick an Isolated one, other workers were applicable for previous versions. A Function project can contain multiple functions that share a common configuration such as environment variables, app settings, and host.
All functions will be deployed together under the same function-app umbrella and will be scaled together.

### Function app files

The default function app consists of:

* `Function.cs` class - Contains a function method with defined triggers and input/output bindings. The `[FunctionName]` attribute marks the method as a function entry point. The name must be unique within a project.
* `local.settings.json` - Contains all app settings, connection strings, and configurations. Settings in the `local.settings.json` file are used only when you're running the project locally so it is not deployed. Note that when you create a new Function project, this file is added to the `.gitignore` file, meaning that it will not end up in the repository if we don't set that up ourselves. By default, it contains the ``AzureWebJobsStorage`` key with a value set to ``UseDevelopmentStorage=true``, which is the connection string for the local Azure Storage Emulator.
* `host.json` - Contains the global config options for all functions within a Function app. Here you can configure logging and retry logic. See options in detail [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json#sample-hostjson-file).
* `Program.cs`  (applicable for out-of-process) - Contains host builder logic, DI configuration and middleware configuration.

### Function Properties

#### Lifetime

Serverless functions should be short-lived and stateless. The maximum lifetime of a function can be configured using the `functionTimeout` property. The default lifetime is 5 and 30 minutes with a maximum of 10 or unlimited for Consumption and other plans respectively. If the function execution time exceeds the configured lifetime it will be un-gracefully killed.

``` json
{
	"functionTimeout": "00:05:00"
}
```

#### Retry logic

[Retry policies](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-error-pages?tabs=csharp#retry-policies-preview) can be defined for all functions in an app by setting properties in host.json or for individual functions using attributes. Retry options are:

- Fixed delay - a specified amount of time is allowed to elapse between each try.
- Exponential backoff - the amount of time between each try increases exponentially

```json
{
	"retry":{
		"strategy":"fixedDelay",
		"maxRetryCount":2,
		"delayInterval":"00:00:03"
	}
}
```

Some of the triggers come with default retry logic. Queue-triggered functions retry 5 times before sending a message to the poison queue, while the timer trigger function doesn't retry at all. When a time trigger function fails, it isn't called again until the next time on the schedule.

#### Triggers

Triggers initialize function invocation while input and output bindings allow input data to be fetched from or pushed to the source without manual integration. They eliminate the implementation of the repetitive code for integration with infrastructure, allowing developers to focus purely on business logic. Available bindings depend on the trigger type we chose. Triggers can be defined by adding the code manually, through the project creation wizard, or the "Add new item" window.

Here are some of the more popular ones:

- **Timer** - Allow running a function by a schedule ( Example: run a function at midnight every night). The schedule format is defined using [NCRONTAB expressions](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=csharp#ncrontab-expressions). This trigger doesn't have any input or output bindings besides the information about when was the task triggered.

```c#
[Function("FunctionName")]
public void Run([TimerTrigger("0 */5 * * * *")] MyInfo myTimer)
{
	_logger.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
	_logger.LogInformation($"Next timer schedule at: {myTimer.ScheduleStatus.Next}");
	// your logic
}
```

- **Queue message / Service bus** - Whenever a message appears on a particular queue/topic, a function is invoked to process the contents of a message. The input parameter of a queue-triggered function is a base64 encoded string message, and there is no output for this function:

```c#
[FunctionName("QueueTrigger")]
public static void Run(
	[QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem,
	ILogger log)
{
	// your logic
}
```

In cases where we need to use a different storage account than other functions in the library, you can use the `StorageAccount` attribute. That attribute specifies the name of the configuration value that contains the storage connection string:

```c#
[StorageAccount("ClassLevelStorageAppSetting")]
public static class AzureFunctions
{
	[FunctionName("QueueTrigger")]
	[StorageAccount("FunctionLevelStorageAppSetting")]
	public static void Run(...)
	{
	// your logic
	}
}
```

- **HTTP request** - Allow using Azure Functions to create web APIs, responding to the various HTTP methods like GET, POST, and PUT. The input parameter is the HTTP request we received, and the output is the response we build.

```c#
[Function("FunctionName")]
public HttpResponseData Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequestData req)
{
	_logger.LogInformation("C# HTTP trigger function processed a request.");
	// your logic
	var response = req.CreateResponse(HttpStatusCode.OK);
	response.Headers.Add("Content-Type", "text/plain; charset=utf-8");
	response.WriteString("Welcome to Azure Functions!");
	return response;
}
```

Note: If there is already an existing App service hosting an API that needs a background job to be executed, it might be a good candidate for an [Azure Web job](https://docs.microsoft.com/en-us/azure/app-service/webjobs-create).

### Middleware

If you want to add middleware to Azure functions, all you have to do is create a new class that inherits from ``IFunctionsWorkerMiddleware`` and register it in your ``HostBuilder``:

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

Now register ``CustomMiddleware`` in the ``Program.cs`` class where you initialized your host builder:

``` c#
var host = new HostBuilder()
    .ConfigureFunctionsWorkerDefaults(
        builder =>
        {
            builder.UseMiddleware<ExceptionLoggingMiddleware>();
        })
    .Build();

host.Run();
```

There is a limitation when using DI in this kind of middleware. You can use constructor injection just like in MVC middleware, but you can't use method injection because the ``Invoke`` method from the ``IFunctionsWorkerMiddleware`` interface accepts only two parameters.

To learn more about Azure function middlewares, read [here](https://learn.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide#middleware).