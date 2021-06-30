Tests in project helps us create stabile applications and we should do them as a part of any project. There are different kind of tests and we mostly write unit and integration tests in .NET development. Tests are always written in a separate project. 
We use [xUnit](https://xunit.net/) testing tool by default, along with [Moq](https://github.com/moq/moq4) for mocking and [Fluent Assertions](https://fluentassertions.com/introduction) for building assertion rules. The tests are structured using the Arrange-Act-Assert pattern which helps us write readable and understandable code.

## Unit Tests

A unit is the smallest logical piece of the application and the unit tests are used to test it out. The tests should be as simple as possible and only cover the functionality of the unit. 

Each test name should contain the name of the method it is testing, a particular condition and the expected result.

```c#
[Fact]
public async Task Process_WhenParameterIsNotValid_ThenThrowArgumentException()
{ }
```

The most used test method is the Fact method which represents one single test with no parameters. But if you want to write tests with parameters and need to test same assertion for multiple different inputs you can always use xUnit's Theory tests. There are multiple ways to pass the data but the most common is the InlineData. 

```c#
[Theory]
[InlineData(-1, 1)]
[InlineData(1, -1)]
public async Task Process_WhenParameterIsNotValid_ThenThrowArgumentException(int a, int b)
{
    // Arrange
    ... 
        
    // Act    
    var result = _sut.Process(a, b);
    Func<Task<Result>> func = () => _sut.Process(a, b);
    
    // Assert
	await func.Should().ThrowAsync<ArgumentException>();
}
```



## Integration Tests

Integration tests allow us to test multiple parts of our system as a group. They are usually a bit more complex and run slower than unit tests. There are many ways to implement them and our approach is the one that covers the whole flow of a certain functionality. 

A good example is testing an API request, from top to bottom. To create a testing environment for our app we can mock the Startup of the app and override the configuration to suit our needs. By using the in-memory database we can run this test anywhere and not have a dependency to a database server. To make the API call, we can prepare the fixture to include a Http client which makes the call. This way we can test the whole flow and its controllers, services, repositories etc.  

This is the example of the test project fixture which we can inject in our test classes and override the setting on the API we are testing: 

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

