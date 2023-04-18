As cloud services became more and more popular, they gave way to a new computing execution model: **serverless**. In this model the cloud provider allocates the necessary resources to run some code, taking care of managing the servers for the job behind the scenes and allowing the developers to focus more on the development of the business logic.

The term "serverless" might look like a misnomer at a first glance - the code is, after all, executed on some kind of a server - but its point is to describe the developer's perspective. Using this model, the developers and DevOps engineers don't have to be concerned with all the usual server maintenance stuff like allocating CPU, memory, and all other resources, managing the network and its security, and planning the capacity, nor do they have to worry about having more resources than the application requires at a specific point in time. The application uses the resources provided by the cloud service only while it is running, meaning that it can be scaled up when needed, or be completely shut down when not in use.

The execution of our code can be triggered by various triggers like an HTTP request, a queue message, changes one the blob storage, or time triggers. All of them will have the same underlying infrastructure and setup, with the only difference being the entry point. 

The main benefits of using serverless solutions are:

* **Writing less code** - The developer can focus on the business logic implementation and utilize the ready-to-use infrastructure features.
* **Maintaining less infrastructure** - Cloud infrastructure providers manage servers, security, and scaling
* **Saving on costs** - No need to buy expensive machines and infrastructure, pay only for the resources that were used by our application.

Even though the benefits might sound like a dream come true, before you start serverless-ing everything from your APIs to your toaster, consider some of the disadvantages of this approach:

- **Coupling with a provider or vendor lock-in** - Serverless applications are highly dependent on the provider which might change the prices, or even end their services altogether. While it is possible to switch between different providers, that process requires at least some kind of development. On the other hand, with the regular hosting models, an application can be switched between servers with only a couple of configuration tweaks.
- **Development and debugging** - Related to the previous point, to run and test the code locally, you usually need to connect to either some kind of an emulator, like Azurite, or directly to the cloud provider, which complicates the setup process for the local development environment. The different execution model also means that some of the debug tools like profilers and debuggers can't be used.
- **Startup performance** - The fact that the application doesn't use any resources while it is not running also implies that it has to go through the startup procedure (almost) every time it needs to run, which impacts the performance. This can be mitigated by using different plans which keep some app instances warm, but that comes at a cost.

### Serverless vs other hosting models

Everything that can be done with serverless computing can also be done with other hosting models. Choosing a hosting model is not a matter of picking what is possible, but rather going down a road that makes the most sense of our application's and the client's requirements. Some clients, like the ones from the banking industry, will require to have all their applications hosted locally for security reasons, while others might want to reduce hosting costs regardless of the potential disadvantages.

All hosting models have their advantages and disadvantages, but we don't necessarily have to fully commit to one of them - we can go with a hybrid approach by picking the right models for different application modules. For example, we could host the API in a container and offload background tasks to serverless functions, allowing communication between them using a message queue.