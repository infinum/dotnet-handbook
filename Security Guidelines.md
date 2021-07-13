In order to pass some of the automated security tests, as well as to keep your .NET application safe it is advised to use some standard security measurements.  This section of handbook will provide insights of some of the most common used strategies.

## User secrets

By right clicking on a project, in Visual Studio, you can select **Manage User Secrets**. It provides an alternative to **appSettings.json** file that overrides its values. Use this to store any type of credentials or sensitive information that is related to your local setup.

User secrets are stored on your local machine and are ignored by Git. Sections that are not provided in user secrets will be taken from **appSettings.json**.

It is strongly advised against pushing any kind of usernames or passwords to Git if possible.

Note that user secrets are not replacement for **appSettings.json** and they should contain all keys required for application to function. Default values for other developers or staging machines should be stored in **appSettings.json**.

Sample of **appSettings.json**:

```json
{
	"ConnectionStrings":{
		"PrimaryDb": "Server=localhost;Port5432;Database:PrimaryDb;User Id=;Password=;"
	},
    "AdditionalSettings":{
        "AuthenticationUrl":"https://this.is.not.sensitive.info.com"
    }
}
```

Sample of **user secrets**:

```json
{
	"ConnectionStrings":{
		"PrimaryDb": "Server=localhost;Port5432;Database:LocalTestDb;User Id=myUsername;Password=myPassword;"
	}
}
```



## CORS

If your application is going to be consumed by a frontend application running on a browser, it is important to setup [CORS](https://www.codecademy.com/articles/what-is-cors) rules. 

If your application is hosted on Azure, you can use cloud built in settings and configuration for CORS found on Azure portal.

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

To enable CORS access for your application, you can use following sample:

**Startup.cs**:

```c#
app.UseCors("AppCors");
```



## Security headers and HTTPS

In order to prevent malicious attacks, it is advised to add additional headers as well as to include to your API responses to keep application secure. A following sample can be used.

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

To restrict application to only be accessible by https calls you can use following sample of code:

**Startup.cs**:

```c#
app.UseHsts();
app.UseHttpsRedirection();
```



## Cookie security

If your application is using cookies, make sure that anti forgery headers are included and that cookie is secure. This is applicable only to MVC apps that have server side rendering. It blocks submitting the form from different domain.

To add anti forgery headers you can use following sample:

**Startup.cs**:

```c#
services.AddAntiforgery(options =>
            {
                options.Cookie.Name = "XSRF-COOKIE";
                options.HeaderName = "X-XSRF-TOKEN";
            });
```

To add a secured cookie to API response you can use following sample:

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

In order to prevent same IP address from targeting a specific end point multiple times in short period, you can use IP rate limiting. It is advised to set this up for all end points that are validating passwords and sensitive information that could potentially under brute force attack.

**AspNetCoreRateLimit** is a library that is most commonly used. 

Once the library is added to the project, following configuration can be used:

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

The library is depended on couple of services and they can be injected in **Startup.cs**. Following code sets up library to use in memory cache to store IP rate policy and rate limit counter. It also uses configuration from *IpRateLimiting* section from **appSettings.json**.

```c#
services.AddSingleton<IIpPolicyStore, MemoryCacheIpPolicyStore>();
services.AddSingleton<IRateLimitCounterStore, MemoryCacheRateLimitCounterStore>();
services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
services.Configure<IpRateLimitOptions>(Configuration.GetSection("IpRateLimiting"));
```

