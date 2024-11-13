ASP.NET Core offers us the built-in health checks Health Checks feature for monitoring and reporting the health and availability of our application infrastructure components. They provide a quick and automated way to verify that our application’s dependencies and components are running smoothly. Health checks help us proactively identify issues within our application, allowing us to address them before they impact users. Regularly verifying our application components' health can ensure a more reliable and resilient system.

## Basic Health Check

For many apps, a basic health probe configuration that reports the app's availability to process requests (liveness) is sufficient to discover the status of the app.

The basic health probe if very simple to implement. All we have to do is register health check services and configure the Health Checks Middleware to respond at a URL endpoint with a health response in `Program.cs`: 

```c#
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddHealthChecks();

var app = builder.Build();

app.MapHealthChecks("/api/health");

app.Run();
```

After adding these, we can access our application on `"/api/health"` and the application will respond with one of the three distinct HealthStatus values:
* HealthStatus.Healthy
* HealthStatus.Degraded
* HealthStatus.Unhealthy

## Custom Health Checks

However, we often need to do more than just confirm the app can respond at the health endpoint URL to make sure it is really healthy, such as checking the database connection or a third party service connection.

For these cases we can create a custom health check by implementing the [IHealthCheck](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.diagnostics.healthchecks.ihealthcheck?view=net-8.0-pp) interface:

```c#
public class SampleHealthCheck : IHealthCheck
{
    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, CancellationToken cancellationToken = default)
    {
        var isHealthy = true;

        // ...

        if (isHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("A healthy result."));
        }

        return Task.FromResult(
            new HealthCheckResult(
                context.Registration.FailureStatus, "An unhealthy result."));
    }
}
```

We can register the health check service by calling `AddCheck` in `Program.cs`: 

```c#
builder.Services.AddHealthChecks()
    .AddCheck<SampleHealthCheck>("Sample");
```

Doing this will configure the health check of the application to return unhealthy in case any of the registered custom checks return unhealthy.

Before you start implementing a custom health check for everything, you should first see if there's already an existing library. In the [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) repository you can find a wide collection health check packages for frequently used services and libraries.
* SQL Server - AspNetCore.HealthChecks.SqlServer
* Postgres - AspNetCore.HealthChecks.Npgsql
* Redis - AspNetCore.HealthChecks.Redis
* RabbitMQ - AspNetCore.HealthChecks.RabbitMQ
* AWS S3 - AspNetCore.HealthChecks.Aws.S3
* SignalR - AspNetCore.HealthChecks.SignalR

```c#
builder.Services.AddHealthChecks()
    .AddCheck<SqlHealthCheck>("custom-sql", HealthStatus.Unhealthy);
    .AddNpgSql(pgConnectionString)
    .AddRabbitMQ(rabbitConnectionString)
```



