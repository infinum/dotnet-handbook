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

The issue here is our static reference to `DateTime.UtcNow`, which returns the current date and time. Unfortunately, despite our incredible technological advancements, we still haven't figured out how to control time. This means that as long as our code is dependent on something as uncontrollable as the unforgiving passage of time, we cannot make it run deterministically.

What we _can_ do is add a layer of abstraction between our code and the way we fetch the current time. This can be done by adding an interface that we can easily mock when setting up our tests:

``` c#
public interface ITimeProvider
{
    public DateTime UtcNow { get; }
}
```

We must also add an implementation of the interface which will be used in the runtime, which is as simple as a service can be:

``` c#
public class TimeProvider : ITimeProvider
{
    public DateTime UtcNow => DateTime.UtcNow;
}
```

In this example, we're keeping it simple by only adding a single property, but we could expand it if we ever needed some other time-related types, like `DateTimeOffset`, or we want to get the time in some other timezone.

Don't forget to register the service in the DI configuration! This service can be registered as a singleton since it only passes a static reference, so there is no need to create more than one instance of it:

``` c#
services.AddSingleton<ITimeProvider>(new TimeProvider());
```

Lastly, we must replace all `DateTime.UtcNow` references with our new interface:

``` c#
private readonly ITimeProvider _timeProvider;

public GreetingGenerator(ITimeProvider timeProvider)
{
    _timeProvider = timeProvider;
}

public string GetTimeBasedGreeting() 
{
    var hourOfTheDay = _timeProvider.UtcNow.Hour;

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
[InlineData(6, 0)]
[InlineData(10, 0)]
[InlineData(11, 59)]
public void GetTimeBasedGreeting_WhenItIsMorning_ThenReturnGoodMorning(
    int hour, 
    int minutes)
{
    _timeProviderMock.Setup(m => m.UtcNow)
        .Returns(new DateTime(2042, 2, 4, hour, minutes, 0));

    var result = _sut.GetTimeBasedGreeting();

    result.Should().BeEquivalentTo("Good morning");
}
```