ASP.NET Core offers the built-in Health Checks feature for monitoring and reporting the health and availability of our application infrastructure components. Health checks provide a quick and automated way to verify that our application’s dependencies and components are running smoothly. They help us proactively identify issues within our application, allowing us to address them before they impact users. Regularly verifying the health of our application components can ensure a more reliable and resilient system.

## Basic Health Check

For many apps, a basic health probe configuration that reports the app's availability to process requests (liveness) is sufficient to discover the app's status.

The basic health probe is very simple to implement. All we have to do is register health check services and configure the Health Checks Middleware to respond at a URL endpoint with a health response in `Program.cs`: 

```c#
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/api/health");

app.Run();
```

After adding these, we can access our application on `"/api/health"` and the application will respond with one of the three distinct `HealthStatus` values:
* `HealthStatus.Healthy`
* `HealthStatus.Degraded`
* `HealthStatus.Unhealthy`

## Custom Health Checks

However, we often need to do more than just confirm the app can respond at the health endpoint URL to make sure it is healthy, such as checking the database connection or a third-party service connection.

In the [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) repository we can find a wide collection of health check packages for frequently used services and libraries, some of which are:
* SQL Server - AspNetCore.HealthChecks.SqlServer
* Postgres - AspNetCore.HealthChecks.Npgsql
* Redis - AspNetCore.HealthChecks.Redis
* RabbitMQ - AspNetCore.HealthChecks.RabbitMQ
* AWS S3 - AspNetCore.HealthChecks.Aws.S3
* SignalR - AspNetCore.HealthChecks.SignalR

We can very simply register any of them in our `Program.cs`: 

```c#
builder.Services.AddHealthChecks()
    .AddSqlServer(sqlServerConnectionString)
    .AddRabbitMQ(rabbitConnectionString);
```

Doing this will configure the health check of the application to return unhealthy in case any of the registered custom checks return unhealthy.

However, for some cases, such as third-party service connections, we will need to write custom health checks. We can do that by  implementing the [IHealthCheck](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.diagnostics.healthchecks.ihealthcheck?view=net-8.0-pp) interface:

```c#
public class RemoteHealthCheck : IHealthCheck
{
    private readonly IHttpClientFactory _httpClientFactory;

    public RemoteHealthCheck(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }

    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken = new CancellationToken())
    {
        using (var httpClient = _httpClientFactory.CreateClient())
        {
            var response = await httpClient.GetAsync("https://api.ipify.org");
            if (response.IsSuccessStatusCode)
            {
                return HealthCheckResult.Healthy("Remote endpoint is healthy");
            }

            return HealthCheckResult.Unhealthy("Remote endpoint is unhealthy");
        }
    }
}
```

and registering it in `Program.cs`: 

```c#
builder.Services.AddHealthChecks()
    .AddCheck<RemoteHealthCheck>("Remote");
```

# Formatting Health Checks Response

By default, the health check endpoint will return a single string value, which isn't very practical if we have multiple health checks configured. We would prefer a more detailed response, containing information about each health check so that we can easily see which services are healthy and which are not. To get this, we would need to provide a `ResponseWriter`, and luckily a good one exists in the AspNetCore.HealthChecks.UI.Client library. 

```c#
app.MapHealthChecks(
    "/health",
    new HealthCheckOptions
    {
        ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
    });
```

This will elevate our health check response from a simple "Unhealthy" to a much more informative response like this one:

```json
{
  "status": "Unhealthy",
  "totalDuration": "00:00:00.0087741",
  "entries": {
    "sqlserver": {
      "data": {

      },
      "duration": "00:00:00.0086714",
      "status": "Healthy",
      "tags": []
    },
    "RemoteHealthCheck": {
      "data": {

      },
      "description": "Remote endpoint is unhealthy",
      "duration": "00:00:00.0002506",
      "status": "Unhealthy",
      "tags": []
    }
  }
}
```



