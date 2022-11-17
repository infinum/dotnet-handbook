In order to pass some of the automated security tests, as well as to keep your .NET application safe it is advised to use some standard security measurements.  This section of the handbook will provide insights into some of the most commonly used strategies.

## CORS

If your application is going to be consumed by a frontend application running on a browser, it is important to setup [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) rules. 

If your application is hosted on Azure, you can use cloud built-in settings and configuration for CORS found on the Azure portal.

To allow all methods and headers, as well as credentials for an array of domains you can use this sample of code:

**Startup.cs**:

```c#
var allowedOrigins = Configuration.GetSection("AllowedOrigins").AsEnumerable();
string[] origins = allowedOrigins.Select(o => o.Value).Where(v => !string.IsNullOrEmpty(v)).ToArray();
services.AddCors(options =>
            {
                options.AddPolicy(AppCors, builder => builder.WithOrigins(origins)
                .AllowAnyMethod()
                .AllowAnyHeader()
                .AllowCredentials());
            });
```

**appSettings.json**:

```json
{
	"AllowedOrigins": [
    	"https://sample.com"
  	]
}
```

To enable CORS access for your application, you can use the following sample:

**Startup.cs**:

```c#
app.UseCors("AppCors");
```



## Security headers and HTTPS

In order to prevent malicious attacks, it is advised to add additional headers as well as to include in your API responses to keep the application secure. The following sample can be used.

**Startup.cs**:

```c#
app.Use(async (context, next) =>
            {
                context.Response.Headers.Add("X-Frame-Options", "deny");
                context.Response.Headers.Add("X-Content-Type-Options", "nosniff");
                context.Response.Headers.Add("X-Xss-Protection", "1");
                context.Response.Headers[HeaderNames.CacheControl] = "no-cache, no-store";
                await next().ConfigureAwait(false);
            });
```

To restrict the application to only be accessible by HTTPS calls you can use the following sample of code:

**Startup.cs**:

```c#
app.UseHsts();
app.UseHttpsRedirection();
```



## Cookie security

If your application is using cookies, make sure that anti-forgery headers are included and that cookie is secure. This applies only to MVC apps that have server side rendering. It blocks submitting the form from a different domain.

To add anti-forgery headers you can use the following sample:

**Startup.cs**:

```c#
services.AddAntiforgery(options =>
            {
                options.Cookie.Name = "XSRF-COOKIE";
                options.HeaderName = "X-XSRF-TOKEN";
            });
```

To add a secured cookie to the API response you can use the following sample:

**SampleController.cs**:

```c#
string accessToken = "this is just a mock data.";
Response.Cookies.Append("token", accessToken, new CookieOptions
            {
                Domain = "infinum.com",
                Secure = true,
                HttpOnly = true,
                SameSite = SameSiteMode.Lax
            });
```



## IP rate limiting

In order to prevent the same IP address from targeting a specific end point multiple times in a short period, you can use IP rate limiting. It is advised to set this up for all end points that are validating passwords and sensitive information that could potentially be under brute force attack.

**AspNetCoreRateLimit** is a library that is most commonly used. 

Once the library is added to the project, the following configuration can be used:

**appSettings.json**:

```json
{
  "IpRateLimiting": {
    "EnableEndpointRateLimiting": true,
    "StackBlockedRequests": false,
    "RealIPHeader": "X-Real-IP",
    "ClientIdHeader": "X-ClientId",
    "HttpStatusCode": 429,
    "QuotaExceededResponse": {
      "Content": "API calls quota exceeded!",
      "ContentType": "application/json",
      "StatusCode": 429
    },
    "GeneralRules": [
      {
        "Endpoint": "*:/api/auth/*",
        "Period": "1m",
        "Limit": 10
      },
      {
        "Endpoint": "*:/api/integrationUser/authenticate/*",
        "Period": "1m",
        "Limit": 10
      }
    ]
  }
}
```

**Startup.cs**

The library is dependent on a couple of services and they can be injected in **Startup.cs**. The following code sets up the library to use in-memory cache to store the IP rate policy and rate limit counter. It also uses configuration from the *IpRateLimiting* section from **appSettings.json**.

```c#
services.AddMemoryCache();
services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
services.Configure<IpRateLimitOptions>(Configuration.GetSection("IpRateLimiting"));
```

To enable IP rate limiting, you can add the following code:

**Startup.cs**:

```c#
app.UseIpRateLimiting();
```
