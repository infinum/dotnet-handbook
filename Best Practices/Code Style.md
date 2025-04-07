It is important to keep the code readable and consistent so anybody reading it can understand it without too much effort. We use general [C# Microsoft coding conventions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions) with some additions to write our code.

These are some of the rules we would like to emphasize:

* Write only one statement per line.
* Write only one declaration per line.
* Add at least one blank line between method definitions and property definitions.
* Line length should not be more than 120 characters.
* Always use braces.
* Format method parameters into separate lines.
* Names for variables and methods should be clear and understandable.
* Boolean variables should be in a form of a question (e.g. isRunning, areFinished, doesExist etc.).
* Keep your methods as simple as possible and break them into multiple methods if they start to gain complexity, or if you can reuse specific parts. If you need comments to describe your method, it may be too complex
* Use file-scoped namespaces (and reduce the indentation level in a whole file). Since there should always be a single namespace in a file, and it's recommended to keep only one class/record per file, there's no need to use the block-scoped namespaces.
* Add sealed modifier wherever possible. When resolving the types of sealed classes, the performance is slightly better because the runtime doesn't have to check whether a class is inherited. Usually, most of the classes/records are not inherited, so this performance improvement could add up throughout the application.

To keep things a bit easier we use [StyleCop](https://github.com/DotNetAnalyzers/StyleCopAnalyzers), a tool for enforcing C# style and consistency rules. The ruleset can be configured and used across all projects in the solution. It is easily installed as a NuGet package and as soon as you build the project, the StyleCop will start checking.

We also use [Roslynator](https://marketplace.visualstudio.com/items?itemName=josefpihrt.Roslynator2019), a lightweight collection of analyzers, refactorings and fixes for C#. Roslynator will be even more useful when the StyleCop can't be added to legacy projects.

You can check out the rulesets we use in our public [GitHub repo](https://github.com/infinum/dotnet-public-content/tree/main/Analyzers). We suggest that instead of copying the files to your repository you use the [`git submodule`](https://git-scm.com/book/en/v2/Git-Tools-Submodules) feature with a symlink to the required directory. That way your repository will stay up to date with a simple `pull` command.

There is also extensive documentation for C# available [here](https://docs.microsoft.com/en-us/dotnet/csharp/).

These are some of the code formatting examples we follow to keep our code clean and consistent:

### Ternary expressions

```c#
// single expression - single line
var result = booleanExpression ? resultA : resultB;

// single expression - multi line
var result = booleanExpression
      ? resultA
      : resultB;

// multiple expressions
var result = booleanExpressionA ? resultA
    : booleanExpressionB ? resultB
    : booleanExpressionC ? resultC
    : resultD;
```



### Lambda expressions

```c#
// single line
var result = data.Where(x => booleanExpression).ToList();

// multi line
var result = data
	.Where(x => booleanExpression)
    .OrderByDescending(x => x.DateTime)
    .FirstOrDefault();

// nested expressions
var result = data
    .Where(x => booleanExpressionA)
    .Select(x => x.ListA
		.Where(y => booleanExpressionB)
		.OrderByDescending(y => y.DateTime)
         .FirstOrDefault())
    .ToList();
```

### Using "Async" suffix in asynchronous method names

In asynchronous programming, one common dilemma is should the "Async" suffix in asynchronous method names be used or not.

On one side, Microsoft recommends using the suffix ([as recommended in this article](https://learn.microsoft.com/en-us/dotnet/csharp/asynchronous-programming/task-asynchronous-programming-model)). On the other hand, the programmers generally tend to write expressive, but as short as possible method names. From that perspective, adding the "Async" suffix may be considered an unnecessary name extension.

We think both views are OK, and thus do not encourage nor discourage any of them. Here, it is important to make naming consistent, at least on the project level. It simplifies the maintenance, and new team members won't be confused when they start working on the project.

However, there is one case in which the "Async" suffix is mandatory. If a method has both a synchronous and an asynchronous version, the asynchronous method needs to be written with the suffix to avoid naming conflict, regardless of the project standard. Following is an example of such a case:

```
	// Synchronous report generation method
    public string GenerateUserReport(int userId)
    {
        // Fetch user data and create report
        var userData = FetchUserData(userId);
        return $"Report for {userData.Name} generated on {DateTime.Now}";
    }

    // Asynchronous report generation method
    public async Task<string> GenerateUserReportAsync(int userId)
    {
        // Simulate an async I/O operation
        var userData = await FetchUserDataAsync(userId);
        return $"Report for {userData.Name} generated on {DateTime.Now}";
    }

    private User FetchUserData(int userId)
    {
        // Example synchronous call; this might be a database or a cache call
        return new User { Id = userId, Name = "John Doe" };
    }

    private async Task<User> FetchUserDataAsync(int userId)
    {
        // Example asynchronous call to simulate delay; could involve an external API or slow data retrieval
        await Task.Delay(100);
        return new User { Id = userId, Name = "John Doe" };
    }
```