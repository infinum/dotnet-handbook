It is important to keep the code readable and consistent so anybody reading it can understand it without too much effort. We use general [C# Microsoft coding conventions](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/inside-a-program/coding-conventions) with some additions to write our code.

These are some of the rules we would like to emphasize:

* Write only one statement per line
* Write only one declaration per line
* Add at least one blank line between method definitions and property definitions
* Line length should not be more than 120 characters
* Always use braces
* Format method parameters into separate lines
* Names for variables and methods should be clear and understandable
* Boolean variables should be in a form of a question (e.g. isRunning, areFinished, doesExist etc.)
* Keep your methods as simple as possible and break them into multiple methods if they start to gain complexity, or if you can reuse specific parts. If you need comments to describe your method, it may be too complex.

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
