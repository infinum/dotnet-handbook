A good project structure makes finding their way around the code easier for developers, especially for the ones which are new to the project. This can be directly translated into quick onboarding times and effortless additions of new features or replacement/fixes of existing ones.

The implementation should be separated from the contract. For example, we can have multiple data sources but the services using them should only be aware of the contract and not the implementation. In case we have multiple data sources, each source should be implemented in a separate project

All the projects inside a solution are separated into two folders (both solution folder and in the file structure), Src and Test. The former contains all the source code which is run in production environment, while the latter contains test projects for each Src project.

This is an example of a project structure that is mostly used for standard .NET Core WebAPIs:

- **Example.API** represents our "presentation" layer. It should contain the endpoints but not the implementation of business logic. Of course, it also contains all the necessary configurations like authorization setup, dependency injection registrations, middleware, and filter registrations.

- **Example.Common** library is used for shared classes and implementations like exception handling, shared models, extensions, etc.

- **Example.Contracts** library contains definitions of all the entities, DTOs, repository interfaces, service interfaces, domain constants, and enumerations.  

- **Example.Services** library contains all the service implementations for the Example application.

- **Example.Data.Db** library contains the database and ORM implementation for the Example application. This project would contain all the database configurations, mappings, migrations, and repository implementation.

- **Example.Data.SomeOtherSource** library should be used if we are using some kind of third-party service to fetch the data. An example would be using Google Drive API to fetch files, which would then be named `Example.Data.Google`.

All the projects that are not used as a "presentation" layer should be implemented as .NET Standard projects. The rest depends on the needs, but in our example, we would use .NET (former .NET Core) for our API.  

#### Test projects

Regardless of the test framework, we try to keep a simple test project structure. For each app project, we create additional test projects. We use simple naming strategies for the projects, only adding the suffix ".Test" for the test project. For example, `Example.API` would be `Example.API.Test`.

You can find more information about app testing in the Testing section of the handbook.

### Configurations

Up until .NET 6, the entry point for adding all configurations and registrations was in the `Startup` class. After that, Microsoft has simplified the initial project structure by having all the code necessary to configure and run the API in `Program.cs`. Even though we could add all our registrations and configurations there, we prefer placing them in extension methods for `IServiceCollection` and `ApplicationBuilder`.

``` c#
namespace Example.Services.Configuration;

public static class ServiceCollectionExtensions
{
    public static IServiceCollection ConfigureExampleServices(this IServiceCollection services)
    {
        services.AddScoped<IExampleService, ExampleService>();
        ...

        return services;
    }
}
```

After that, we can simply call the methods from our `Program.cs`:

``` c#
using Example.Services.Configuration;

...

builder.Services.ConfigureExampleServices();

```

Those extension methods should be placed in the project that contains the code which the method configures. For example, methods for registering all our services to DI container should be placed in `Example.Services` project, while confguring database-related code should be placed inside `Example.Data.Db`. 

If we have direct project dependencies within our solution, we can add the dependant configuration method call inside another configuration method. For example, if our services project depends on some interfaces from `Example.Data.AzureStorage`, then we can call that configuration method from the Services project's configuration method:

``` c#

using Example.Data.AzureStorage.Configuration;

namespace Example.Services.Configuration;

public static class ServiceCollectionExtensions
{
    public static IServiceCollection ConfigureExampleServices(this IServiceCollection services)
    {
        
        ...
        // Calling a method from Example.Data.AzureStorage:
        services.AddAzureDataServices();
        ...

        return services;
    }
}
```

This way we ensure that all the required services are registered with just one method call.

If you want to know more about managing configurations, you can check out our [Configuration section](../Api/configuration).

