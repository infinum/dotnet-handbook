API documentation is a manual on how to use and integrate with the API. It consists of all the information required to work with it, usually consisting of endpoint descriptions with their parameters and return values, but can also include tutorials and examples as well. It should be simple and easily understandable, mainly meant for other developers that need to use the API. Although API documentation is very important, it should never be considered to be the full documentation of a certain system, as it only describes the communication 

This helps us a lot when collaborating with other teams since it provides an excellent interface to API. In our APIs, we mostly use [Swagger](https://swagger.io/) toolset to automatically generate documentation which enables us to quickly update it whenever something changes.

Swagger uses OpenAPI specification for its generated documentation which is the industry standard for describing REST APIs. Using it allows both computers and humans to understand available endpoints of a REST API without access to the source code. 

### Using Swagger

To use Swagger you  need to install one of the NuGet packages that support it and configure it for your needs. In their documentation, Microsoft suggests using two packages - [Swashbuckle](https://github.com/domaindrivendev/Swashbuckle.AspNetCore) or [NSwag](https://github.com/RicoSuter/NSwag). In the example below, we will focus on configuration for Swashbuckle, but other package have a similar principle.

The configuration consists of two parts. The first one is the configuration of a Swagger service which configures document information. That can be specified in the service configuration part of the `Program.cs` (or in older versions in `Startup.ConfigureServices()`): 

```c#
builder.Services.AddSwaggerGen(options => 
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

The second part of the configuration is done in the app configuration part of `Program.cs` (formerly in `Startup.Configure()`) where we set up and enable Swagger middleware that will serve generated Swagger JSON. Here we can also configure the endpoint for SwaggerUI. By default, Swagger will be served at the root of the URL.

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

It is recommended to move the configuration to a separate project so it can be used by multiple projects. It can be done by implementing extension methods for `IServiceCollection` and `IApplicationBuilder`.


#### XML documentation

Swagger has the ability to use the project's generated documentation file to add more information about the endpoints displayed on the SwaggerUI page.

First step is to tell the compiler to build the XML file when building the API. We do that by adding the following lines to the `.csproj` file of the API:

``` XML
<PropertyGroup>
    ...
    <GenerateDocumentationFile>true</GenerateDocumentationFile>
    <NoWarn>$(NoWarn);1591</NoWarn>
    ...
</PropertyGroup>
```

The first line commands the compiler to build the XML file. If we only add that line, then the analyzer will add warnings to all the classes, fields, and methods which don't have the XML comments. Since we only want to add the comments on controllers, we can remove those warnings with the `<NoWarn>` tag.

After that, we need to tell Swagger to use the generated XML file. That is done by expanding the configuration:

``` c#
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new OpenApiInfo { Title = "Example.Api", Version = "v1" });
    var xmlFilename = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    c.IncludeXmlComments(Path.Combine(AppContext.BaseDirectory, xmlFilename));
});
```

And that's it, the setup is done. Now all you have to do is add the comments on the endpoints, describing the endpoint functionality and all the parameters. We recommend using the following XML sections:

- `<summary>` - a short description of the endpoint's functionality
- `<remarks>` - some additional information about the endpoint. This is displayed in the endpoint descrption only after it's expanded, meaning that here we can add more details about the endpoint (e.g. description of the flow, available enum values, different behaviours for different user roles, etc.).
- `<param>` - description of a specified parameter.
- `<returns>` - description of the returned value.

Having those sections in mind, a fully commented could look something like this:

```c#
/// <summary>
/// Gets a paged list of important stuff.
/// </summary>
/// <remarks>
/// Regular users will only get the most important stuff, admins will 
/// get all the important stuff.
/// </remarks>
/// <param name="pageSize">Size of the page.</param>
/// <param name="pageNumber">Number of the page.</param>
/// <returns>List of important stuff.</returns>
[HttpGet]
public async Task<IActionResult> GetImportantStuffList(
    [FromQuery] int pageSize = 10,
    [FromQuery] int pageNumber = 1)
{
    ...
}
```

... and that should produce the following endpoint description (return models are cut for brevity):

![Endpoint with comments](/resources/endpoint-with-comments.png)

#### Endpoint results

By default, Swagger will expect that every endpoint will only return an OK result with an empty body. Since in most of our endpoints that will not be the case, we have to define all the results that could be produced by that endpoint. We can do that by using Data Annotation attributes:

- **`[ProducesResponseType]`** - specifies what HTTP response codes will be returned with corresponding models (if needed).
- **`[ProducesErrorResponseType]`** - specifies the default error model for all the error response codes (4xx and 5xx). Just have in mind that this only defines the response model, not the response codes - those should still be defined using the `[ProducesResponseType]` attribute.

These attributes can be placed on both method and class level, which applies them on a certain endpoint and controller (meaning all the endpoints inside the controller) respectively.

Having these attributes in mind, an endpoint described like this:

```c#
[ProducessErrorResponseType(typeof(ErrorResponseDto))]
[ProducesResponseType(StatusCodes.Status401Unauthorized)]
public class ExampleController : ControllerBase
{
    ...

    [HttpGet]
    [ProducesResponseType(typeof(DownloadReportResponse), StatusCodes.Status200OK)]
    public async Task<IActionResult> GetImportantStuffList(
        [FromQuery] int pageSize = 10,
        [FromQuery] int pageNumber = 1)
    {
        ...
    }
}
```

... will produce response type definitions that look something like this:

![Endpoint return values](/resources/swagger-return-models.png)
