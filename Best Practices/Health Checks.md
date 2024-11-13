ASP.NET Core offers us Health Checks Middleware and libraries for monitoring and reporting the health and availability of our application infrastructure components. They provide a quick and automated way to verify that our application’s dependencies and components are running smoothly. Health checks help us proactively identify issues within our application, allowing us to address them before they impact users. Regularly verifying our application components' health can ensure a more reliable and resilient system.

## Basic health probe

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

