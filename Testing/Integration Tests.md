Integration tests allow us to test multiple parts of our system as a group. They are usually a bit more complex and run slower than unit tests. There are many ways to implement them and our approach is the one that covers the whole flow of certain functionality.

A good example is testing an API request, from top to bottom. To create a testing environment for our app we can mock the Startup of the app and override the configuration to suit our needs. By using the in-memory database we can run this test anywhere and not have a dependency on a database server. To make the API call, we can prepare the fixture to include an HTTP client which makes the call. This way we can test the whole flow and its controllers, services, repositories, etc.  

This is the example of the test project fixture which we can inject into our test classes and override the setting on the API we are testing: 

```c#
public class TestServerFixture : IDisposable
{
    public TestServer Server { get; }
    public HttpClient Client { get; }

    public TestServerFixture()
    {
        var builder = new WebHostBuilder()
            .UseEnvironment("Development")
            .ConfigureAppConfiguration((hostContext, config) =>
			{
                var env = hostContext.HostingEnvironment;
                config.Sources.Clear();
                config.AddJsonFile("appsettings.json", optional: true);
                config.AddJsonFile($"appsettings.{env.EnvironmentName}.json", optional: true);
                config.AddUserSecrets<FakeStartup>();
            })
            .UseStartup<FakeStartup>()
            .UseSetting(
            	WebHostDefaults.ApplicationKey,
            	typeof(FakeStartup).GetTypeInfo().Assembly.GetName().Name);

        Server = new TestServer(builder);
        Client = Server.CreateClient();
        Client.BaseAddress = new Uri("https://localhost:5000");
    }
}
```
