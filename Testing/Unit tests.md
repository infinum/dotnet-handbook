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
