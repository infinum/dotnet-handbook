Add as a new project, through VS Function App template. This creates azure function app that can contain multiple functions that share common configuration such as environment variables, app settings and host.
All functions will be deployed together under same function-app umbrella and  scaled together.

### Triggers

To trigger function execution, different triggers can be used. Most popular ones are :

* Timer - Allow running a function by a schedule ( Example: run a function at midnight every night). Schedule format and samples can be found [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=csharp#ncrontab-expressions).

```c#
		[Function("FunctionName")]
		public void Run([TimerTrigger("0 */5 * * * *")] MyInfo myTimer)
		{
		    _logger.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
		    _logger.LogInformation($"Next timer schedule at: {myTimer.ScheduleStatus.Next}");
		    // your logic
		}
```

* Queue message / Service bus trigger - Whenever message appears on a particular queue/topic function is invoked to process the contents of message. Input parameter of queue triggered function is base64 encoded string message.

```c#
		[FunctionName("QueueTrigger")]
		public static void Run(
		    [QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem,
		    ILogger log)
		{
		   // your logic
		}
```

or with storage account settings configured as attribute

```c#

		[StorageAccount("ClassLevelStorageAppSetting")]
		public static class AzureFunctions
		{
		    [FunctionName("QueueTrigger")]
		    [StorageAccount("FunctionLevelStorageAppSetting")]
		    public static void Run( //...
		    {
			// your logic
		    }
		}
```

* HTTP trigger - Allow using Azure Functions to create web APIs, responding to the various HTTP methods like GET, POST, and PUT

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

### Function app files

Default function app consists of:

* **Function.cs** class - Contains function method with defined trigger and input output bindings. ``[FunctionName]`` attribute marks the method as a function entry point. Name must be unique within a project.
* **local.settings.json** - Contains all app settings, connectionstrings and configurations needed for the function development. Settings in the local.settings.json file are used only when you're running project locally so it is not deployed. By default, it contains  ``AzureWebJobsStorage`` key with a value set to ``UseDevelopmentStorage=true``, which is the connection string for the local Azure Storage Emulator.
Azure Functions are interlocked with Azure Storage services, meaning that every Azure function needs Azure storage configured to be able to run. The storage account connection is used by the Functions host for operations such as managing triggers and logging function executions. It's also used when dynamically scaling function apps.
* **host.json** - Contains the global config options for all functions within a Function app. Here you can configure logging, retry logic. See options in detail [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json#sample-hostjson-file).
* **Program.cs**  (applicable for out-of-process) - Contains host builder logic, DI configuration, middleware configuration.

### Best practices

* Function should be short-lived, so whenever possible, refactor large functions into smaller function sets that work together and return responses fast.

* Functions should be stateless and idempotent if possible. Associate any required state information with your data. A function could process data based on that state while the function itself remains stateless.

* Function could encounter an exception at any time. Design your functions with the ability to continue from a previous fail point during the next execution.

* Plan scaling, it is important to understand how your function app responds to load and how the triggers can be configured to handle incoming events.

* Newly built functions should be out-of process.
