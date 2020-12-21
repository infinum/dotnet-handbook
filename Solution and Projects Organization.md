### Project structure

A good project structure makes getting around the code easier, especially to developers that just joined the project. This also makes adding new features or replacing/fixing existing ones.

The implementation should be separated from the contract. For example we can have multiple data sources but the services using them should only be aware of the contract and not the implementation. In multiple data sources case, different data sources should be implemented in separate projects. 

This is an example of a project structure that is mostly used for standard .NET Core WebAPIs:

* Example.API
* Example.Common
* Example.Contracts
* Example.Services
* Example.Data.Db
* Example.Data.SomeOtherSource (if there is a need for one)

Example.API represents our "presentation" layer. It should contain the endpoints but not the implementation of business logic. Of course it also contains all the necessary configurations like authorization setup, dependency injection registrations, middleware and filter registrations.

Example.Common library is used for shared classes and implementations like exception handling, shared models, extensions etc. 

Example.Contracts library represents all the entities, DTOs, repository interfaces, service interfaces, domain constants and enumerations.  

Example.Services library contains all the service implementations for the Example application. 

Example.Data.Db library contains the database and ORM implementation for the Example application. This project  would contain all the database configurations, mappings, migrations and repository implementation. 

Example.Data.SomeOtherSource library should be used if we are using some kind of third party service to fetch the data. A good example is using Google Drive API to fetch files and of course then the project would be named Example.Data.Google. 

All the projects that are not used as a "presentation" layer are implemented as a .NET Standard projects. The rest depends on the needs but in our Example, we would use .NET Core for our API.  



#### Startup.cs

This is where the most of the configuration for the application is. We use it to register services, repositories, data base context, etc. Depending on the application type we can also tell our app to use middlewares, filters, or any other third party service.

To keep things organized it is a good practice to group specific configuration sections like services and repositories. One of the ways to do this is through implementation an IServiceBuilder extension. 

Since we use [Options pattern](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.1) for app configuration the bindings are done in the startup class. There are more details for this pattern in the Design pattern section of this handbook. 



#### AppSettings.json

Appsettings.json file is usually used for app configuration like connection strings, configurable parameters, etc. Since is not a good practice to keep app secrets here and upload them to repository we mostly use environment variables for configuring the app in non-development environment. In development environment we  use [user secrets](https://docs.microsoft.com/en-us/aspnet/core/security/app-secrets?view=aspnetcore-3.1&tabs=windows). 



#### NuGet

It is a good practice to keep app packages updated, and for installing and updating we use NuGet package manager. 

#### Test projects

Regardless of test framework we are using, we try to keep simple test project structure. For each app project we create additional test projects. We use simple naming strategies for the projects, only adding the suffix ".Test" for the test project. For example:

Example.API would be Example.API.Test

You can find more information about app testing in the Testing section of the handbook.
