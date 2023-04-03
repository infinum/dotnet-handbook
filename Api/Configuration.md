Configuration is the set of external parameters provided to an application that controls the application’s behavior in some way. It typically consists of a mixture of settings and secrets that the application will load at runtime.

The configuration system in ASP.NET Core is very flexible, allowing you to load the configuration from a wide range of locations. Also, ASP.NET Core provides the ability to easily bind this configuration to strongly typed options objects, which then you can inject into your services. This model of strongly typed configuration makes it easy to logically group settings around a given feature and lends itself well to unit testing.

The ASP.NET Core configuration model centers on two main constructs:

- `ConfigurationBuilder` - describes how to construct the final configuration representation for your app
- `IConfiguration` - holds the configuration values themselves.

### Configuration providers

ASP.NET Core uses [configuration providers](https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration-providers) to load key-value pairs from a variety of sources. Those configuration providers can load values from files (like `appsettings.json` or the user secrets file), environment variables, command-line arguments, a database, or a remote service. You can even create your custom configuration provider if none of the above satisfy your needs.

We use the `CreateDefaultBuilder` helper method to bootstrap `HostBuilder` for our app. The `CreateDefaultBuilder` method calls `ConfigureAppConfiguration` and sets up a number of default configuration providers.

### appsettings.json

`appsettings.json` file is usually used for app configuration like connection strings, configurable parameters, etc. The configurations from this file will be used in all the environments, regardless if the application is built in debug or release modes. 

But what if we want to have some configuration values that should be used only during the development time? For that exact purpose we can use `appsettings.Development.json`. This file is used only when running the API in debug mode, so we don't have to worry about misconfiguring our app in other environments. 

The above-mentioned files are a part of the source code, meaning that they will end up in the repository. While some configuration values are not secret (e.g. task execution timings, read batch sizes...), there are almost awlways some values which should not be made public (e.g. connection strings, client secrets, ...). Because of that, we mostly use environment variables and cloud secrets managers for configuring the APIs in non-development environments, and [user secrets](../best-practices/user-secrets) in development environments.

### Options pattern

ASP.NET Core promotes the use of strongly typed settings as a way of letting configuration code adhere to the single responsibility principle and allowing the injection of configuration classes as explicit dependencies. Such settings also make testing easier. Instead of having to create an instance of `IConfiguration` to test a service, you can create an instance of the POCO options class.

Your options classes need to be non-abstract and have a public parameterless constructor to be eligible for binding. The binder will set any public properties that match configuration values!

To help facilitate the binding of configuration values to your custom options classes, ASP.NET Core introduces the `IOptions<T>` interface. This is a simple interface with a single property, Value, which contains an instance of your configured options class at runtime. To use the interface, we must first register our configuration class:


``` c#
services.Configure<ExampleSettings>(
	configuration.GetSection("ExampleSettings"));
```

If you want to know more about where to place the configuration registrations, check out the [Project structure](../project-organization/project-structure) section of this handbook.

After the registration, the configuration can be injected into services using Dependency Injection by resolving the `IOptions<T>` service:

``` c#
...
private readonly ExampleSettings _options;

public ExampleService(IOptions<ExampleSettings> options)
{
	_options = options.Value;
}
...
```

### Reloading configurations

Reloading is generally only available for file-based configuration providers. The `Add*File` extension methods (used for reading various file formats, just replace the "*" with the extension you need) include an overload with a `reloadOnChange` parameter. If this is set to true, the app will monitor the filesystem for changes to the file and will trigger a complete rebuild of the `IConfiguration`, if necessary.

``` c#
public static void AddAppConfiguration(
	HostBuilderContext hostingContext,
	IConfigurationBuilder config)
{
	config.Add*File(
		"exampleAppSettings.*",
		optional: true
		reloadOnChange: true);
}
```

**Disclaimer:** The `IOptions<T>` interface is registered in the DI container as a singleton, so if you need reloading functionality you should use the [`IOptionsSnapshot<T>` interface](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-7.0#use-ioptionssnapshot-to-read-updated-data).

### Documentation

If you want to know more visit these links:

- [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
- [Options pattern in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
