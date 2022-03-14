## Versioning

Using explicit route based API versioning client is always aware of what version of the API is being used and it allows the backend to maintain a backwards compatible endpoints.

Our policy is to always embedded version string in the path of the request URL, at the end of the service root: eg. `https://service.infinum.com/api/v1.0/`

### Using `Microsoft.AspNetCore.Mvc.Versioning` package for API versioning

The following snippet needs to be added as a part of `ConfigureServices` step, to configure versioning library :

```c#
    services.AddApiVersioning(
        options =>
        {
            options.ApiVersionReader = new UrlSegmentApiVersionReader();
        });
```

Note : there's no need to configure default API version or fallback resolution, we are always going to use explicit API route segment version reader.

For each endpoint route you need to add `api/v{version:apiVersion}` and need to tag which versions the controller supports with `[ApiVersion("version")]` attribute(s) :

```C#
[Route("api/v{version:apiVersion}/example")]
[ApiVersion("1")]
[Produces("application/json")]
[ApiController]
public class ExamplesController : ControllerBase
{
    ...
}
```

#### Implementing multiple API versions in the same controller

```C#
[Route("api/v{version:apiVersion}/example")]
[ApiVersion("1")]
[ApiVersion("2")]
[Produces("application/json")]
[ApiController]
public class ExamplesController : ControllerBase
{
    [HttpGet]
    public string ExampleBoth => "same in both versions";

    [MapToApiVersion("1")]
    [HttpGet("result")]
    public string ExampleV1() => "result v1";

    [MapToApiVersion("2")]
    [HttpGet("result")]
    public string ExampleV2() => "result v2";
}
```

Implementing API versioning in code highly dependent on existing codebase - it can be as simple as adding a minor revision in the same controller or as complex as having separate namespaces for each version on controller/DTO/service layer.

#### Deprecating a Service Version

```C#
[Route("api/v{version:apiVersion}/example")]
[ApiVersion("1", Deprecated = true)]
[Produces("application/json")]
[ApiController]
public class ExamplesController : ControllerBase
{
    ...
}
```

Deprecation warning will be shown in the documentation generated and can be included in the generated HTTP headers.
To include versioning headers with each response set `ReportApiVersions` to true in `AddApiVersioning` configuration :

```C#
    services.AddApiVersioning(
        options =>
        {
            options.ReportApiVersions = true;
            options.ApiVersionReader = new UrlSegmentApiVersionReader();
        });
```

#### Additional documentation

Additional information on API versioning with `Microsoft.AspNetCore.Mvc.Versioning` can be found here : https://github.com/dotnet/aspnet-api-versioning/wiki/How-to-Version-Your-Service

### Configuring Swashbuckle for route based API versioning

`Microsoft.AspNetCore.Mvc.Versioning` will add versioning support to Swashbuckle but it will include parameter based versioning.

To exclude parameter based routing add the following operation filter to Swashbuckle :

```c#
public class RemoveVersionFromParameter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var versionParameter = operation.Parameters.FirstOrDefault(p => p.Name == "version");
        if (versionParameter is not null)
        {
            operation.Parameters.Remove(versionParameter);
        }
    }
}
```

Services configuration snippet :

```c#
        services.AddSwaggerGen(
            c =>
            {
                c.SwaggerDoc(
                    "v1",
                    new OpenApiInfo
                    {
                        Title = "Template API",
                        Version = "1",
                    });

                // add other document versions

                c.OperationFilter<RemoveVersionFromParameter>();
            });
```
