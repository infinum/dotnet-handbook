## Azure Functions

Azure Functions is a Function-as-a-Service (“FaaS”), i.e. a platform for running configurable blocks of code that run in response to some event.
Functions support several different programming languages such as C# and JavaScript, F#, Java, Python, and PowerShell.

There are three basic hosting plans available for Azure Functions: 
* Consumption plan

Scale automatically and only pay for compute resources i.e. when functions are running. Cold start is startup latency, so first trigger execution after function was idle takes looger time. 

* Premium plan

Automatically scales based on demand using pre-warmed workers which run applications with no delay after being idle, runs on more powerful instances, and connects to virtual networks.
No cold start.

* Dedicated (App Service) plan.

Run functions within an App Service plan at regular App Service plan rates.
Sutable for long-running scenarios such as Durable Functions, or where predictable costs are needed. 

Note: If there is already existingg App service hosting some API that needs some background job to be executed good candidate is Azure Web job.
To learn more, see [here](https://docs.microsoft.com/en-us/azure/app-service/webjobs-create)

#### Scaling rules 
 A single function app in Consumption plan, only scales out to a maximum of 200 instances.
 Single instance may process more than one message or request at a time though, so there isn't a set limit on number of concurrent executions.
 Timer trigger functions are singleton (trigger only one instance of function)

Azure Functions are a serverless solution that provide:
* Writing less code 
Allows developer to focus on the business logic implementation and utilize the infrastructure ready to use features. 

* Maintain less infrastructure 
Cloud infrastructure providers manage servers, security and scaling. Functions are scaled automatically.

* Save on costs 
No need to buy resources in advance - pay as you go.

#### Function properties
* Short lived 
Serverless functions should be short-lived and stateless. Default lifetime is 5 minutes with maximum of 10 minutes. If the function execution time exceeds the configured lifetime it will be un-gracefully killed.

* Retry logic 
Retry policies can be defined for all functions in an app or for individual functions by setting properties in host.json that will affect all functions or using attributes.
Retry options are : fixedDelay or ExponentialBackoff. Learn more [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-error-pages?tabs=csharp#retry-policies-preview)
Note: Some of the trrigers can come with default retry logic like queue triggered functios, the timer trigger doesn't retry after a function fails. When a function fails, it isn't called again until the next time on the schedule.
*
Triggers and Bindings
Triggers initialize function invocation while input and output bindings allow input data to be fetched from or pushed to the source without manual integration.
They eliminate repetitive code for integration with infrastructure, allowing development to focus purely on business logic.


## Set up 
Add as a new project, through VS Function App template. This creates azure function app that can contain multiple functions that share common configuration such as environment variables,app settings and host.
All functions will be deployed together under same functions app umbrella, scaled together. ( Single instance of a function's runtime will host all the functions in function app)
Different triggers and binndings are avaliable :


#### Triggers
To trigger function execution, different triggeres can be used. Most popular ones are : 
* Timer
Allow to running a function by a schedule ( Example: run a function at midnight every night)
Schedule format and samples can be found [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-timer?tabs=csharp#ncrontab-expressions)

* Queue message / Service bus trigger
Whenever message appears on a particular queue/topic function is invoked and process the contents of message. 

* HTTP trigger 
Allow using Azure Functions to create web APIs, responding to the various HTTP methods like GET, POST, and PUT


Default function app consists of :
* Function.cs class 
Contains function method with defined trriger and input output bindings.
FunctionName attribute marks the method as a function entry point. Name must be unique within a project.

* local.settings.json 
Contains all app settings, connection string and settings needed in function development. By default, it contains 
AzureWebJobsStorage key with a value set to UseDevelopmentStorage=true, which is the connection string for the local Azure Storage Emulator. 
Azure Functions are interlocked with Azure Storage services, meaning that every Azure function needs Azure storage configured to be able to to run.
Azure Storage account keeps track of function execution state and triggers.

* host.json 
Contains the global config options for all functions within a Function app. Here you can configure logging, retry logic.
See options in detail [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-host-json#sample-hostjson-file)

* Program.cs  (applicable for out-of-proccess)
Contains host builder logic, DI configuration, middleware configuration. 


#### In Process

The Azure Functions host is the runtime for .NET functions. C# function apps have run in the same process as the host. Sharing a process has enabled unique benefits to .NET functions, most notably is a set of rich bindings and SDK injections. 
It was possible to bind directly to the SDKs and types used in the host process—including advanced types like CloudBlockBlob from the Storage SDK or Message from the Service Bus SDK.
However, as side effect, dependencies could conflict (like Newtonsoft.Json) and running in the same process also means that the .NET version of user code must match the .NET version of the host.

####  Out Of Process
Out of process is running runctions as isolated process from the main Azure Functions host. 
This allows Azure Functions to rapidly support new versions of these languages without updating the host, and it allows the host to evolve independently to enable new features and update its own dependencies over time.

#### Differences
* Out of process function dependeny injection is done via usual .Net core pattern while in-process need custom StartUp class.
* Out of process function  supports middleware configuration.
* In process functions support rich binding classes while out-of-process binding classes are simple DTOs, strings byte[].

For more info read [here](...)


.NET 6 functions support both in-process and isolated process options, but out-of-process does not support all bindings and features supported in in-process.
In .Net 7 out of process will support full set of fetures avaliable currently in-process. From .Net 7 out of process will be the only supported option.
To find out more read [here](https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-roadmap/ba-p/2197916)
