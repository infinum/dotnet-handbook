## Azure Functions

Azure Functions is a Function-as-a-Service (“FaaS”), i.e. a platform for running configurable blocks of code that run in response to some event.
Functions support several different programming languages such as C# and JavaScript, F#, Java, Python, and PowerShell.

In general, Azure Functions are a serverless solution that provide:
* Writing less code - Allows developer to focus on the business logic implementation and utilize the ready-to-use infrastructure features. 
* Maintain less infrastructure - Cloud infrastructure providers manage servers, security and scaling. Functions are scaled automatically.
* Save on costs - No need to buy resources in advance - pay as you go.


There are three basic hosting plans available for Azure Functions: 
* Consumption plan - Functions scale automatically and billing is calculated only for compute resources used i.e. when functions are running. Cold start is startup latency, so first trigger execution after function was idle takes logger time. 
* Premium plan - Automatically scale using pre-warmed workers which run applications with no delay after being idle. Function runs on more powerful instances, and connects to virtual networks. No cold start.
* Dedicated (App Service) plan. - Run functions within an App Service plan at regular App Service plan rates. This plan is suitable for long-running scenarios such as Durable Functions, or where predictable costs are needed. 

Note: If there is already existing App service hosting some API that needs some background job to be executed, good candidate can be Azure Web job.
To learn more, see [here](https://docs.microsoft.com/en-us/azure/app-service/webjobs-create).


#### Scaling 
 A single function app in Consumption plan, scales out to the maximum of 200 instances, while a single instance can process more than one message / request at a time.
 
Timer trigger functions are singleton (trigger only one instance of function).

In the real world scenarios we need to control scaling of instances. Sometimes azure functions deal with databases or other services which may have scaling limits so over-scaling azure functions could put other resources under pressure. Scaling rules can be configured for function depending on trigger type. 
 
Example: If for some reason, we need one queue message at the time to be processed by queue triggered function than, we need to configure function app scale-out to one function instance. Than the function concurent proccessing should be limited too, and that is done by setting ``batchSize`` to 1 in the host.json. Learn more [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#concurrency) and [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-concurrency).
 

#### Function properties
* Short lived - Serverless functions should be short-lived and stateless. Default lifetime is 5 minutes with maximum of 10 minutes. If the function execution time exceeds the configured lifetime it will be un-gracefully killed.
* Retry logic - Retry policies can be defined for all functions in an app by setting properties in host.json or for individual functions using attributes.
Retry options are : ``fixedDelay`` or ``ExponentialBackoff``. Learn more [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-error-pages?tabs=csharp#retry-policies-preview).



		{
		   "retry":{
		      "strategy":"fixedDelay",
		      "maxRetryCount":2,
		      "delayInterval":"00:00:03"
		   }
		}
   
   

Note:  Some of the triggers come with default retry logic. Queue triggered functions retry 5 times berfore sending message to poison queue, while the timer trigger function doesn't retry at all. When a time trigger function fails, it isn't called again until the next time on the schedule.

* Triggers and Bindings - Triggers initialize function invocation while input and output bindings allow input data to be fetched from or pushed to the source without manual integration. They eliminate implementation of the repetitive code for integration with infrastructure, allowing development to focus purely on business logic.


#### Function set up 

Add as a new project, through VS Function App template. This creates azure function app that can contain multiple functions that share common configuration such as environment variables, app settings and host.
All functions will be deployed together under same function-app umbrella and  scaled together.

#### Triggers
To trigger function execution, different triggers can be used. Most popular ones are : 
* Timer - Allow running a function by a schedule ( Example: run a function at midnight every night). Schedule format and samples can be found [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=csharp#ncrontab-expressions).


		[Function("FunctionName")]
		public void Run([TimerTrigger("0 */5 * * * *")] MyInfo myTimer)
		{
		    _logger.LogInformation($"C# Timer trigger function executed at: {DateTime.Now}");
		    _logger.LogInformation($"Next timer schedule at: {myTimer.ScheduleStatus.Next}");
		    // your logic 
		}


* Queue message / Service bus trigger - Whenever message appears on a particular queue/topic function is invoked to process the contents of message. Input parameter of queue triggered function is base64 encoded string message.


	
		[FunctionName("QueueTrigger")]
		public static void Run(
		    [QueueTrigger("myqueue-items", Connection = "StorageConnectionAppSetting")] string myQueueItem, 
		    ILogger log)
		{
		   // your logic 
		}
  
  
  

or with storage account settings configured as attribute 


	
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


* HTTP trigger - Allow using Azure Functions to create web APIs, responding to the various HTTP methods like GET, POST, and PUT


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




#### Function app files 
Default function app consists of :
* Function.cs class - Contains function method with defined trigger and input output bindings. ``[FunctionName]`` attribute marks the method as a function entry point. Name must be unique within a project.
* local.settings.json - Contains all app settings, connectionstrings and configurations needed for the function development. Settings in the local.settings.json file are used only when you're running project locally so it is not deployed. By default, it contains  ``AzureWebJobsStorage`` key with a value set to ``UseDevelopmentStorage=true``, which is the connection string for the local Azure Storage Emulator. 
Azure Functions are interlocked with Azure Storage services, meaning that every Azure function needs Azure storage configured to be able to run. The storage account connection is used by the Functions host for operations such as managing triggers and logging function executions. It's also used when dynamically scaling function apps. 
* host.json - Contains the global config options for all functions within a Function app. Here you can configure logging, retry logic. See options in detail [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json#sample-hostjson-file).
* Program.cs  (applicable for out-of-process) - Contains host builder logic, DI configuration, middleware configuration. 

 
#### Host Configuration

* In-Process - Azure Functions host is the runtime for .NET functions. Before .Net 5 C# function apps run in the same process as the host. Sharing a process has enabled unique benefits to .NET functions, most notably is a set of rich bindings and SDK injections. However, as side effect, dependencies could conflict (like Newtonsoft.Json) and running in the same process also means that the .NET version of user code must match the .NET version of the host.
* Out Of Process - Out of process is running functions as isolated process from the main Azure Functions host. This allows Azure Functions to rapidly support new versions of these languages without updating the host, and it allows the host to evolve independently to enable new features and update its own dependencies over time.

Differences:

* Dependency injection for out-of-process functions is done via usual .Net core pattern while in-process need custom StartUp class.
* Out of process function supports middleware configuration.
* In process functions support rich binding classes while out-of-process binding classes are simple DTOs, strings, byte[].

For more info read [here](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=browser&pivots=development-environment-vs#differences-with-net-class-library-functions).

.NET 6 functions support both in-process and out-of-process options, but out-of-process does not support all bindings and features supported in in-process.
In .Net 7 out-of-process will support full set of features so out-of-process will be the only supported option. To find out more read [here](https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-roadmap/ba-p/2197916).

#### Best practices 
* Function should be short-lived, so whenever possible, refactor large functions into smaller function sets that work together and return responses fast.

* Functions should be stateless and idempotent if possible. Associate any required state information with your data. A function could process data based on that state while the function itself remains stateless.

* Function could encounter an exception at any time. Design your functions with the ability to continue from a previous fail point during the next execution.

* Plan scaling, it is important to understand how your function app responds to load and how the triggers can be configured to handle incoming events. 

* Newly built functions should be out-of process.


