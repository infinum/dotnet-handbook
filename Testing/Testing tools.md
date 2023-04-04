As it is with a lot of things in software development, when writing tests, we don't want to reinvent the wheel for each project. The .NET ecosystem has a lot of 1st and 3rd party tools available that can make our lifes easier when writing unit tests (and no, we're not thinking of just asking ChatGPT to write them for you). Here are just some of them that we use regularly.

### Testing framework

Most popular testing frameworks for .NET are [NUnit](https://nunit.org/), [xUnit](https://xunit.net/) and [MSTest](https://learn.microsoft.com/en-us/dotnet/core/testing/unit-testing-with-mstest). We use xUnit by default because it encourages clean code, reduces the need for code duplication and is simpler to use than other testing frameworks.

### Test parameters

When we want to test a piece of code with different parameters, but similar expected results, we could write a single test case for each parameter value. Although this would work, this kind of code duplication should immediately cause a twitch in your eye. There must be a better way to define those tests, right? Test methods are, after all, methods, so why don't we just pass the differences as parameters and let the method define the testing logic arround them.

xUnit supports a few ways of defining the test parameters. When a test method is decorated with the `[Theory]` attribute, the test framework knows that there will be more attributes which define the various test parameters.

#### Inline Data

Using the `[InlineData]` attribute is the simplest way to define the test parameters. We repeat the attribute for each set of parameters, and the test framework then knows to repeat that test for each set:

``` c#
[Theory]
[InlineData("")]
[InlineData("  ")]
[InlineData(null)]
public void Validate_WhenNameIsNullOrEmpty_ThenReturnFalse(string name)
{
    var user = new AutoFaker<User>()
        .RuleFor(u => u.Name, _ => name)
        .Generate();

    var result = _userValidator.Validate(user);

    result.Should().Be(false);
}
```

#### Member Data

Just as you got rid of that eye twitch by using `[InlineData]`, you start writing another test method using the same approach. You add the `[Theory]` attribute, start adding the parameters and suddenly that twitch returns. Are we really adding the same parameters again? Have no fear, the `[MemberData]` attribute is here!

This attribute allows us to define a static property in the test class that contains the list of test data which can be used as test parameters for multiple test methods:

``` c#
public static IEnumerable<object[]> EmptyStrings =>
    new List<object[]>
    {
        new object[] { "" },
        new object[] { " " },
        new object[] { default(string) },
    };

[Theory]
[MemberData(nameof(EmptyStrings))]
public void Validate_WhenAddressIsEmpty_ThenReturnFalse(string address)
{
    var user = new AutoFaker<User>()
        .RuleFor(u => u.Address, _ => address)
        .Generate();
    
    ...
}
```

You might have noticed that the attribute reads a list of object arrays. This feature allows us to define multiple parameters for each test case. Just be mindful of the order of the parameters - the values you define in the array will be passed in the same order that you define them in.

#### Class Data

With all the itches scratched and twitches sorted out, you go on with writing the tests for a different class. Equipped with the test parameter knowledge you start writing another test, but just as you've defined the first two parameters, that twitch returns yet again. "Wait," you think to yourself, "I've seen the same parameters in that previous class. There must be a way to avoid this duplication once again!"

Luckily for you, there are solutions for that as well. The `[ClassData]` attribute allows us to reference a class which define a list of test parameter sets. First, we must define that class. The class must implement the `IEnumerable<object[]>` interface by defining what exact data will be returned:

``` c#
public class EmptyStringTestData
    : IEnumerable<object[]>
{
    public IEnumerator<object[]> GetEnumerator()
    {
        yield return new object[] { "" };
        yield return new object[] { " " };
        yield return new object[] { default(string) };
    }

    IEnumerator IEnumerable.GetEnumerator() => GetEnumerator();
}
```

Just as it was the case with `MemberData`, the list we're working with is returning an array of object, allowing us to define a set of parameters for each test case.

After we've defined the class with the parameters, we must tell the testing framework to use them by referencing the class:

``` c#
[Theory]
[ClassData(typeof(EmptyStringTestData))]
public void Validate_WhenProductNameIsEmpty_ThenReturnFalse(string name)
{
    var product = new AutoFaker<Product>()
        .RuleFor(p => p.Name, _ => name)
        .Generate();
    ...
}
```



### Generating test data

When it comes to generating test data, we use two main approaches: data generators and static mock methods. 

#### Data generators

By default, we use data generators because they are much easier to set up and maintain for simple test cases. Here our weapon of choice is [AutoBogus](https://github.com/nickdodd79/AutoBogus). We use it because it has a lot of powerful options when it comes to specifying rules for specific properties, but also it is very simple if we need to generate an object with filled properties:

``` c#
User user = new AutoFaker<User>();
```

In case we need to set specific values on some properties, we can use the `RuleFor()` method:

``` c#
User user = new AutoFaker<User>()
    .RuleFor(e => e.Name, f => f.Name.FullName());
```

When we're setting rules like these, we don't want to repeat ourselves for each test case. To avoid repetition, we can set the generator as a private field, and then use it wherever we need it:
``` c#
public class UserServiceTests
{
    private readonly Faker<User> _userFaker = new AutoFaker<User>
        .RuleFor(e => e.Name, f => f.Name.FullName());

    [Fact]
    public void GetUser_WhenUserExists_ReturnUserFromDb()
    {
        var expectedUser = _userFaker.Generate();
        ...
    }
}
```

One important thing to note with data generators is that they generate **random** data every time we run a test. This might sound obvious to you, but it is also a fact that can be easily overlooked. If we don't define the rules properly, we might end up with random test failures when the generator returns some data which breaks some ouf our business rules we weren't expecting to be broken. This can be quite hard to debug because in that process we must encounter the same (or similar) random case which causes the test failure. When using data generators we always must ask ourselves what exactly are we testing, what are our valid data states, and what restrictions we must define.

#### Static methods

There are some situations in which we need data in some specific format with a lot of rules. At some point defining those rules for data generators becomes too complicated or even impossible. In these cases, we default to writing our own generator methods where we define every single property of a class ourselves.

``` c#
public static class UserMock
{
    public static User Build()
    {
        return new User()
        {
            Name = "Test User",
            Address = "Testing street 1a",
            LastLoginDate = new DateTime(2022, 2, 22, 14, 2, 22),
        };
    }
}
```

This approach gives us the flexibility to define the test data exatly how we want it. Of course, there is a downside to this approach: defining every property can be cumbersome, especially when dealing with complex classes, and maintaining the generator methods can take a lot of time. Because of that we use static methods only in cases where data generators don't work for us.

### Mocking

Now that we know how to generate test parameters and test data, we must set them up in our test cases. This is where mocking frameworks come in. For this purpose we use [Moq](https://github.com/moq/moq4), a powerful mocking framework that allows us to set up behaviours and verify usage of objects.

Moq allows us to define mock objects by wrapping the classes we must use, but are not a part of our test. This approach enables us to skip the logic defined in that method and just define the result we would expect in our test case. First, we must define that wrapper:

``` c#
var someServiceMock = new Mock<SomeService>();
```

After the wrapper is defined, we must pass the mocked object to the class or method we're testing:

``` c#
var sut = new ServiceWeAreTesting(someServiceMock.Object);
```

The `ServiceWeAreTesting` doesn't know that the object we passed to it is a mocked one. From its perspective, it is just a regular object whose methods and properties can be called, and they will behave as expected. But that behaviour doesn't just magically come out of the box - it is something we must define in our Setup process.

#### Setup

In the setup process we must define two parts: what property or method will be called, and what will be the result of that call. Here is an example of a simple setup of a calculator service:

``` c#
calculatorServiceMock
    .Setup(m => m.Add(2, 3))
    .Returns(5);
```

In this setup we have defined that if someone calls the `Add()` method with parameters 2 and 3, it will return the result 5. But what if someone calls it with a different set of parameters? Since we haven't defined that behaviour, the mocked service will return `null`.

What if we want to specify a return value for any parameter that was given to the method? For these cases, Moq provides the `It` class, which has a couple of useful methods. First we will look at the `IsAny<T>()` method. This method is a replacement for all possible values:

``` c#
calculatorServiceMock
    .Setup(m => m.Add(It.IsAny<int>(), It.IsAny<int>()))
    .Returns(5);
```
This setup will return the result 5 for regardless of the input parameters. In some cases we might want to add some restrictions to the input parameters. To help us with that, we can use the `.Is<T>()` method, which takes an expression as a parameter which is used to validate the input. As an example, let's say that our test method returns a certain value only for positive integers:

``` c#
calculatorServiceMock
    .Setup(m => m.Add(It.Is<int>(x => x > 0), It.Is<int>(x => x > 0)))
    .Returns(5);
```

When using these parameter definitions, it is a good practice to define the return value for as minimal set of input parameters as possible. This means that we should avoid using `It.IsAny<T>()` if we have any restrictions or expectations for our input parameters, or even avoid using `It.Is<T>` if we have the exact value we will be expecting.

Sometimes that is easier said than done. If we are dealing with complex objects which are generated inside the method we're testing, we can't simply compare it to an expected value, and the expression for validating the whole object could be a large one. In those cases we can use the `Callback()` method which allows us to do something with the input parameters. In this example, we will extract the value the method received, and then use Fluent Assertions (more on this in the section below) to validate the input:

``` c#
ComplexRequest expectedInput = ComplexRequestMock.BuildExpectedInput();
ComplexRequest receivedInput = null;

complexServiceMock
    .Setup(m => m.DoComplexWork(It.IsAny<ComplexRequest>()))
    .Callback(input => receivedInput = input);

receivedInput.Should().BeEquivalentTo(expectedInput);
```

This approach is especially useful when a method is called multiple times, and we need to do something with each individual input.

Now that we've defined how exactly will the method be called, we must define what value will be returned (if the method or a property is expected to return the value). In the examples above, we've used a single returning value, but that is not the only way to define a return value. We can use the input parameters to determine it:

``` c#
calculatorServiceMock
    .Setup(m => m.Add(It.IsAny<int>(), It.IsAny<int>()))
    .Returns<int, int>((x, y) => x + y);
```

#### Verifications

Another powerful and useful feature of the mocking framework is the ability to validate that a certaing method or property was called. This can be done using the `Verify()` method:

``` c#
calculatorServiceMock
    .Verify(m => m.Add(2, 3), Times.Once());
```

This example verifies that the `Add()` method was called exactly once with the parameters 2 and 3. If the conditions are not met, an exception will be thrown which will cause the test to fail.

When defining these verifications, the same rule for argument specificity should be applied: we should allow the test to pass with as minimal set of input arguments as possible. This, again, means that we should avoid using `It.IsAny<T>()` if we can be more specific. The only exception to this rule is when we are validating that a certain method was never called. In those cases we should cover all the possible values:

``` c#
calculatorServiceMock
    .Verify(m => m.Add(It.IsAny<int>(), It.IsAny<int>()), Times.Never());
```

### Asserting

The last step of a test is asserting that the code is behaving as expected. Besides validating the method calls we mentioned in the section above, we often must validate the return value of a method or property. Even though the xUnit framework provides the `Assert` class with static methods for comparing various values, we tend to use [FluentAssertions](https://fluentassertions.com/) for its advanced features and readability.

Most common use case for assertions is to check that the returned object of a method contains the expected data. For that we need to construct an equivalent object with expected values and compare it to the returned value:

``` c#
var expectedResult = ResultMock.BuildExpected();
var result = sut.DoSomethingImportant();

result.Should().BeEquivalentTo(expectedResult);
```

This method will throw an exception if any of the properties in the two objects are not the same. Just have in mind that only the primitive values are compared by values, while others will be compared by reference. If we want to ignore some properties in this comparison, we can do that too:

``` c#
result.Should().BeEquivalentTo(
    expectedResult,
    options => options.Excluding(x => x.IrrelevantProperty)
);
```

Another important thing to note is that the `IsEquivalentTo()` method will compare all the properties only if the object it is comparing doesn't have the `Equals()` method overriden. If it does, it will use that method regardless of the comparison rules we've defined.