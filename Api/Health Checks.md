ASP.NET Core offers the built-in Health Checks feature for monitoring and reporting the health and availability of our application infrastructure components. Health checks provide a quick and automated way to verify that our application’s dependencies and components are running smoothly. They help us proactively identify issues within our application, allowing us to address them before they impact users. Regularly verifying the health of our application components can ensure a more reliable and resilient system.

## Basic Health Check

For many apps, a basic health probe configuration that reports the app's *availability to process requests* (liveness) is sufficient to discover the app's status.

The basic health probe is very simple to implement. All we have to do is register health check services and configure the Health Checks Middleware to respond at a URL endpoint with a health response in `Program.cs`: 

```c#
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/api/health");

app.Run();
```

When the basic infrastructure for health checks is set up, the application will expose a health check endpoint (`"/api/health"`) that can be queried to determine the overall health status. If the application is up and running, the health check will respond with HTTP status code 200 OK and a response `"Healthy"`. If the application encounters a failure to respond (e.g., due to crashes or hosting issues), the health endpoint won't be reachable, and external monitoring systems will interpret this as "unhealthy."

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

Doing this will configure the health check of the application to recognize the application as unhealthy in case any of the registered health checks return `"Unhealthy"`.

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

## Health Check Response

Based on the state of the app, the health check endpoint will respond with one of the three distinct `HealthStatus` values:

* `HealthStatus.Healthy`
* `HealthStatus.Degraded`
* `HealthStatus.Unhealthy`

The `HealthStatus.Healthy` status indicates that the application or resource is functioning as expected and by default returns the HTTP status code 200 OK.
The `HealthStatus.Degraded` status suggests that the health checks did succeed, but there might be some performance issues or non-critical problems. For example, the degraded status could be used for a simple database query that did succeed but took more than a second. This status by default also returns the HTTP status code 200 OK.
The `HealthStatus.Unhealthy` status indicates a critical failure in the application or its dependencies, as well as an unhandled exception being thrown while executing the health check. This health status by default returns the HTTP status code 503 Service Unavailable.

With the basic setup, the health check endpoint will return a single string value, which isn't very practical if we have multiple health checks configured. We would prefer a more detailed response, containing information about each health check so that we can easily see which services are healthy and which are not. To get this, we would need to provide a `ResponseWriter`, and luckily a good one exists in the AspNetCore.HealthChecks.UI.Client library. 

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



