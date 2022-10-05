Configuration is the set of external parameters provided to an application that controls the application’s behaviour in some way. It typically consists of a mixture of settings and secrets that the application will load at runtime.

The configuration system in ASP.NET Core is very flexible, allowing you to load configuration from a wide range of locations. Also, ASP.NET Core provides the ability to easily bind this configuration to strongly typed options objects, which then you can inject into your services. This model of strongly typed configuration makes it easy to logically group settings around a given feature and lends itself well to unit testing.

The ASP.NET Core configuration model centers on two main constructs:

- `ConfigurationBuilder` - describes how to construct the final configuration representation for your app
- `IConfiguration` - holds the configuration values themselves.

If you want to know more about how to manage configurations during the development, check out our [User secrets](best-practices/user-secrets) section.

### Configuration providers

ASP.NET Core uses configuration providers to load key-value pairs from a variety of sources. Applications can use many different configuration providers. You can load values from files, environment variables, command-line arguments, from a database, from a remote service, or you could create your own custom configuration provider.

We use the `CreateDefaultBuilder` helper method to bootstrap `HostBuilder` for our app. The `CreateDefaultBuilder` method calls `ConfigureAppConfiguration` and sets up a number of default configuration providers.

### Options pattern

ASP.NET Core promotes the use of strongly typed settings as a way of letting configuration code adhere to the single responsibility principle and to allow the injection of configuration classes as explicit dependencies. Such settings also make testing easier. Instead of having to create an instance of `IConfiguration` to test a service, you can create an instance of the POCO options class.

Your options classes need to be non-abstract and have a public parameterless constructor to be eligible for binding. The binder will set any public properties that match configuration values!

To help facilitate the binding of configuration values to your custom options classes, ASP.NET Core introduces the `IOptions<T>` interface. This is a simple interface with a single property, Value, which contains instance of your configured options class at runtime. Option settings are configured in the `ConfigureServices` section of `Startup`:

``` c#
	---
	services.Configure<ExampleSettings>(
		configuration.GetSection("ExampleSettings"));
	---
```

Configuration can now be injected into services using Dependency Injection by resolving the `IOptions<T>` service:

``` c#
	---
	private readonly ExampleSettings _options;
	public ExampleService(IOptions<ExampleSettings> options)
	{
		_options = options.Value;
	}
	---
```

### Reloading configurations

Reloading is generally only available for file-based configuration providers. The `Add*File` extension methods (used for reading various file formats, just replace the "*" with the extension you need) include an overload with a `reloadOnChange` parameter. If this is set to true, the app will monitor the filesystem for changes to the file and will trigger a complete rebuild of the `IConfiguration`, if necessary.

``` c#
	---
	public static void AddAppConfiguration(
		HostBuilderContext hostingContext,
		IConfigurationBuilder config)
	{
		config.Add*File(
			"exampleAppSettings.*",
			optional: true
			reloadOnChange: true);
	}
	---

```

**Disclaimer:** The `IOptions<T>` interface is registered in the DI container as a singleton, so if you need reloading functionality you should use the `IOptions-Snapshot<T>` interface.

### Documentation

If you want to know more visit these links:

- [Configuration in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/)
- [Options pattern in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options)
