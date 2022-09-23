* In-Process - Azure Functions host is the runtime for .NET functions. Before .Net 5 C# function apps run in the same process as the host. Sharing a process has enabled unique benefits to .NET functions, most notably is a set of rich bindings and SDK injections. However, as side effect, dependencies could conflict (like Newtonsoft.Json) and running in the same process also means that the .NET version of user code must match the .NET version of the host.
* Out Of Process (Isolated) - Out of process is running functions as isolated process from the main Azure Functions host. This allows Azure Functions to rapidly support new versions of these languages without updating the host, and it allows the host to evolve independently to enable new features and update its own dependencies over time.

Differences:

* Dependency injection for out-of-process functions is done via usual .Net core pattern while in-process need custom StartUp class.
* Out of process function supports middleware configuration.
* In process functions support rich binding classes while out-of-process binding classes are simple DTOs, strings, byte[].

For more info read [here](https://docs.microsoft.com/en-us/azure/azure-functions/dotnet-isolated-process-guide?tabs=browser&pivots=development-environment-vs#differences-with-net-class-library-functions).

.NET 6 functions support both in-process and out-of-process options, but out-of-process does not support all bindings and features supported in in-process.
In .Net 7 out-of-process will support full set of features so out-of-process will be the only supported option. To find out more read [here](https://techcommunity.microsoft.com/t5/apps-on-azure-blog/net-on-azure-functions-roadmap/ba-p/2197916).
