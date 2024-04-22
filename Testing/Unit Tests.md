The first type of automated tests most software developers encounter are unit tests. That's because the unit tests are an excellent tool for determining whether a specific code change or a new implementation caused some unexpected behaviors.

The idea behind unit testing is to split the code into the smallest logical chunks - called _units_ - and test their functionalities separately. This means that the unit tests should be as simple as possible and cover only the functionality of that unit.

Their simplicity makes running all unit tests in a project quick and easy, which is why we should be checking them periodically during development time and especially before publishing our branch and creating a PR. We can go even further with that idea - by integrating test runs in the CI/CD pipelines and PR merge requirements we cannot forget to run them before sharing the code with others!

Detailed unit tests help us validate that our code snippets do what they were meant to do, but they won't validate that we've implemented the requested functionalities successfully. To do that, all our units must work together, which is why we have other types of tests like [integration](integration-tests) and [UI](ui-tests) tests.

### Tips and tricks

#### Testing the current time

Let's say that we have to write unit tests for the following method:

``` c#
public string GetTimeBasedGreeting() 
{
    var hourOfTheDay = DateTime.UtcNow.Hour;

    if (hourOfTheDay >= 6 && hourOfTheDay < 12)
        return "Good morning";
    
    if (hourOfTheDay >= 12 && hourOfTheDay < 18)
        return "Good afternoon";

    return "Good evening";
}
```

It is easy to think of the test cases for this: we must check for each time of the day that the correct greeting is returned. But as soon as we start writing the tests, we come to an issue: only one greeting can be tested at a certain point in time. This means that we have no way of defining unit tests that will pass every time we run them, even though our code is working perfectly!

The issue here is our static reference to `DateTime.UtcNow`, which returns the current date and time. 

From .NET 8 Microsoft introduced `TimeProvider` abstract class that should replace our custom layer of abstraction for making our code more testable. Previously, we would have something like `IClockProvider` that we would register in the DI container and inject in our service classes.

Since the introduction of `TimeProvider` in .NET 8 we get all that out of the box. All we need to do is register the default implementation of the `TimeProvider` as a singleton and inject it in our service and replace direct use of `DateTime.UtcNow`. Default implementation uses current system clock.

```c#
builder.Services.AddSingleton(TimeProvider.System);
```

``` c#
private readonly TimeProvider _timeProvider;

public GreetingGenerator(TimeProvider timeProvider)
{
    _timeProvider = timeProvider;
}

public string GetTimeBasedGreeting() 
{
    var hourOfTheDay = _timeProvider.GetUtcNow().Hour;

    if (hourOfTheDay >= 6 && hourOfTheDay < 12)
        return "Good morning";
    
    if (hourOfTheDay >= 12 && hourOfTheDay < 18)
        return "Good afternoon";

    return "Good evening";
}
```

And that's it, with this simple change we can make this method think it is any time of the day we want:

``` c#
[Theory]
[InlineData(11, 59, "Good morning")]
[InlineData(12, 0, "Good afternoon")]
[InlineData(18, 01, "Good evening")]
public void GetTimeBasedGreeting_WhenTimeAvailable_ThenReturnAppropriateGreeting(
    int hour, 
    int minutes,
    string expectedGreeting)
{
    _timeProvider
        .GetUtcNow()
        .Returns(new DateTime(2042, 2, 4, hour, minutes, 0));

    var result = _sut.GetTimeBasedGreeting();

    result.Should().BeEquivalentTo(expectedGreeting);
}
```

TimerProvider class enables us to do much more than just work with current time, so for more examples check out this [article](https://andrewlock.net/exploring-the-dotnet-8-preview-avoiding-flaky-tests-with-timeprovider-and-itimer/).
