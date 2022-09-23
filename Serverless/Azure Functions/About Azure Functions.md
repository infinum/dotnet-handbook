Azure Functions is a Function-as-a-Service (“FaaS”), i.e. a platform for running configurable blocks of code that run in response to some event.
Functions support several different programming languages such as C# and JavaScript, F#, Java, Python, and PowerShell.

Main benefits of using Azure Functions and serverless solutions are:

* Writing less code - Allows developer to focus on the business logic implementation and utilize the ready-to-use infrastructure features.
* Maintain less infrastructure - Cloud infrastructure providers manage servers, security and scaling. Functions are scaled automatically.
* Save on costs - No need to buy resources in advance - pay as you go.


There are three basic hosting plans available for Azure Functions:

* Consumption plan - Functions scale automatically and billing is calculated only for compute resources used i.e. when functions are running. Cold start is startup latency, so first trigger execution after function was idle takes longer time.
* Premium plan - Automatically scale using pre-warmed workers which run applications with no delay after being idle. Function runs on more powerful instances, and connects to virtual networks. No cold start.
* Dedicated (App Service) plan. - Run functions within an App Service plan at regular App Service plan rates. This plan is suitable for long-running scenarios such as Durable Functions, or where predictable costs are needed.

Note: If there is already existing App service hosting some API that needs some background job to be executed, good candidate can be Azure Web job.
To learn more, see [here](https://docs.microsoft.com/en-us/azure/app-service/webjobs-create).


### Scaling

 A single function app in Consumption plan, scales out to the maximum of 200 instances, while a single instance can process more than one message / request at a time.

Timer trigger functions are singleton (trigger only one instance of function).

In real world scenarios we need to control scaling of instances. Sometimes azure functions deal with databases or other services which may have scaling limits so over-scaling azure functions could put other resources under pressure. Scaling rules can be configured for function depending on trigger type.

Example: If for some reason, we needed queue messages to be processed by a function one at a time, than we need to configure function app scale-out to one function instance. Than the function concurent proccessing should be limited too, and that is done by setting ``batchSize`` to 1 in the host.json. Learn more [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-storage-queue-trigger?tabs=csharp#concurrency) and [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-concurrency).


### Function properties

* Short lived - Serverless functions should be short-lived and stateless. Default lifetime is 5 minutes with maximum of 10 minutes. If the function execution time exceeds the configured lifetime it will be un-gracefully killed.
* Retry logic - Retry policies can be defined for all functions in an app by setting properties in host.json or for individual functions using attributes.
Retry options are : ``fixedDelay`` or ``ExponentialBackoff``. Learn more [here](https://docs.microsoft.com/en-us/azure/azure-functions/functions-bindings-error-pages?tabs=csharp#retry-policies-preview).

```json
               {
		   "retry":{
		      "strategy":"fixedDelay",
		      "maxRetryCount":2,
		      "delayInterval":"00:00:03"
		   }
		}
```

Note:  Some of the triggers come with default retry logic. Queue triggered functions retry 5 times before sending message to poison queue, while the timer trigger function doesn't retry at all. When a time trigger function fails, it isn't called again until the next time on the schedule.

* Triggers and Bindings - Triggers initialize function invocation while input and output bindings allow input data to be fetched from or pushed to the source without manual integration. They eliminate implementation of the repetitive code for integration with infrastructure, allowing development to focus purely on business logic.
