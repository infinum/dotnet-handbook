### API Documentation

API documentation is basically a manual how to use and integrate with some API. It consists of all the information required to work with it usually including tutorials and examples as well. It should be simple and easily understandable so anyone who needs to use it can with as little time possible find what they need. 

For us it helps us with collaborating with other teams. In .NET Core applications the tool we use for automatical generation of documentation is [Swagger](https://swagger.io/) which enables us to quickly update the documentation whenever something changes.

#### Swagger

Swagger uses OpenAPI specification for it's generated documentation which is the industry standard for describing REST APIs. Using it allows both computers and humans to understand available endpoints of a REST API without access to the source code. 

To use Swagger it is enough to install one of the NuGet packages that support it and configure it for your needs. For example main 2 that Microsoft suggests using in their documentation are [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) or [NSwag](https://github.com/RicoSuter/NSwag). In the example we will be showing configuration with Swashbuckle, but others have similar principle.

For it work as expected there are 2 things that we have to configure. The first one is configuration of a swagger service which configures document information and it is done in `Startup.ConfigureServices()`: 

```c#
services.AddSwaggerGen(options => 
{
	options.SwaggerDoc("version", new OpenApiInfo
    {
	   	Version = "version",
        Title = "title",
        Description = "description",
        ...
    })   
})
```

Second part of configuration is done in `Startup.Configure()` where we setup and enable Swagger middleware that will serve generated swagger JSON. Here we can also setup endpoint for swagger UI which can be customized by providing CSS file location. By default Swagger will be served at the root of the URL.

```c#
// This enables middleware for swagger JSON document
// By providing options parameter we can setup some settings, for example URL prefix
app.UseSwagger(opt => 
{
    opt.RouteTemplate = = $"{prefix}/{{documentname}}/swagger.json";
});

// This enables SwaggerUI, SwaggerEndpoint setting must match endpoint URL for swagger setup above
app.UseSwaggerUI(opt => 
{
    opt.SwaggerEndpoint("/swagger/v1/swagger.json", "name");
});
```

If possible it is recommended to move configuration to separate project so it can be used by multiple projects. It can be done by extending `IServiceCollection` and `IApplicationBuilder`.

#### Customization

To further describe API endpoints and models we can use attributes from `System.ComponentModel.DataAnnotations` namespace. These can provide information on what they can expect from the response to anyone who uses the API.

```c#
// Example of some attributes that can be applied
[Produces("application/json")]
[ProducesResponseType(StatusCodes.Status200Ok)]
[ProducesResponseType(StatusCodes.Status400BadRequest)]
...
```

