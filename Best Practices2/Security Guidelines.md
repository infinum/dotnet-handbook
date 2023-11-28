During the development of an application, besides worrying about writing clean code with good performance, we must always keep an eye on security. After all, the fastest and prettiest code is not that useful if allows malicious actors to access our users' data. This section of the handbook will provide insights into some of the most commonly used strategies.

## HTTPS

When it comes to API communication, the first obvious step in improving security is to allow communication only through HTTPS. This protocol encrypts the request and response contents, which makes reading their contents difficult even if an unwanted actor intercepts them. We can enforce these rules with a couple of simple method calls which add middleware that will block or redirect any request that is not using HTTPS:

```c#
app.UseHsts();
app.UseHttpsRedirection();
```

## CORS

[**C**ross-**o**rigin **r**esource **s**haring](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) is a mechanism that controls access to resources based on the domain a request is coming from. This mechanism allows us to block requests from unknown sources.

The first step of the setup process is defining what domains we want to allow. We advise setting those values in the app configuration because it allows us to define different values for different environments:

```json
{
	"AllowedOrigins": [
    	"https://sample.com"
  	]
}
```

Now that we've defined that, pass these values to the CORS configuration. To do that, we can implement an extension method:

```c#
public static void SetupCorsRules(
    this IServiceCollection services, 
    IConfiguration configuration)
{
    var allowedOrigins = configuration
        .GetSection("AllowedOrigins")
        .AsEnumerable()
        .Select(o => o.Value)
        .Where(v => !string.IsNullOrEmpty(v))
        .ToArray();

    services.AddCors(options =>
    {
        options.AddPolicy(
            "OurImportantCorsPolicy", 
            builder => builder.WithOrigins(allowedOrigins)
                .AllowAnyMethod()
                .AllowAnyHeader()
                .AllowCredentials());
    });
}
```

Lastly, we will add a method call for the setup and for using the defined policy in `Program.cs`:

``` c#
...
builder.Services.SetupCorsRules(builder.Configuration)
...
var app = builder.Build();
...
app.UseCors("OurImportantCorsPolicy");
```

With this configuration, our application will only allow requests coming from `https://sample.com`.

Note that similar CORS configurations are available for many services we use, like Azure Blob Storage. These should also be set up to prevent malicious users from accessing our data on other services.

## Security headers

There are a couple of attacks that can be prevented by adding a few security headers. Here are some that we recommend using:

- [`X-Frame-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options) - indicates whether or not a browser should be allowed to render a page in a frame object, which is used to avoid click-jacking attacks by ensuring that their content is not embedded into other sites.
- [`X-Content-Type-Options`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Content-Type-Options) - indicates that the MIME types advertised in the `Content-Type` headers should be followed, which is used to avoid MIME-type sniffing.
- [`X-XSS-Protection`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-XSS-Protection) - a feature of some browsers that stops pages from loading when they detect reflected cross-site scripting (XSS) attacks. Note that this header is non-standard and is not on a standards track, meaning that it will not work for every user. This is also largely unnecessary in modern browsers with implementations of content security policies.
- [`Cache-Control`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control) - controls caching in browsers and shared caches. By setting the value to `no-cache`, we instruct the cache to always validate the content with the origin server before returning it to the requester, which ensures that the requester will always get the newest version of the content.

We can add these headers to all our responses by adding a middleware that will add the headers to every response:

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

When using these headers, we must have in mind the fact that they are only instructions for the browsers. This means that we're depending on the "goodwill" of browser developers to implement the mechanisms we advise using. Since most of our users will use one of the more popular browsers which do support these headers, this will not be an issue for them (and us), but these headers do not guarantee protection like other security guidelines which are implemented on the server side.

## Cookie security

If your application is using cookies, make sure that anti-forgery headers are included and that cookie is secure to prevent malicious attacks. This applies only to MVC apps that have server-side rendering. It blocks submitting the form from a different domain.

### Anti-forgery tokens

If a malicious site configures sending a request to a different site, the browser will automatically add the site's cookies to the request, making the request seem legitimate to the server. This kind of attack is called a [CSRF](https://owasp.org/www-community/attacks/csrf) attack. To prevent that, we can use anti-forgery tokens, additional tokens generated by the server and placed in a hidden form field. The token provided by the server must be included in a request for submitting the form.

To add anti-forgery headers you can use the following sample:

```c#
services.AddAntiforgery(options =>
    {
        options.Cookie.Name = "XSRF-COOKIE";
        options.HeaderName = "X-XSRF-TOKEN";
    });
```

### Secure cookies

Enabling secure cookies tells the browser that the cookie should only be transmitted using Transport Layer Security (TLS). This way the attackers can't sniff the network and read the cookie from outgoing requests.

To add a secured cookie to the API response we can use the following sample in our controller methods:

```c#
string accessToken = "this is not hardcoded, believe me.";
Response.Cookies.Append("token", accessToken, new CookieOptions
    {
        Domain = "infinum.com",
        Secure = true,
        HttpOnly = true,
        SameSite = SameSiteMode.Lax
    });
```

## IP rate limiting

Endpoints that validate passwords and other sensitive information can be susceptible to brute-force attacks. To prevent them, or at least slow them down up to the point that renders them useless, we can add IP rate limiting. This feature blocks requests from a specific IP address if the number of requests in a certain period exceeds our defined threshold (e.g. if it sends more than 10 requests in 30 seconds).

There are many ways to set this up. For simpler use cases, we could add this feature to our app using the [`Microsoft.AspNetCore.RateLimiting`](https://learn.microsoft.com/en-us/aspnet/core/performance/rate-limit?view=aspnetcore-7.0) middleware or a package like [`AspNetCoreRateLimit`](https://github.com/stefanprodan/AspNetCoreRateLimit). This approach will stop the request on the app level, but most of our projects will have some infrastructure around the app which can do the same thing.

If our app is hosted on Azure, we can configure [request throttling](https://learn.microsoft.com/en-us/azure/api-management/api-management-sample-flexible-throttling) or, in case we are implementing microservice architecture, [rate limiting for Azure Front Door Service](https://learn.microsoft.com/en-us/azure/web-application-firewall/afds/waf-front-door-rate-limit). In more traditional setups, we may have reverse proxies like Nginx in our system, which can be [configured to do the same thing](https://www.nginx.com/blog/rate-limiting-nginx/). We advise talking to the DevOps engineers about what's the best way to configure this on our system.