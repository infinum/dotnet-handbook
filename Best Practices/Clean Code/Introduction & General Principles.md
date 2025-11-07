### What is the clean code ?


![cleanCodeImg](/resources/wtf.png)


The image above is a pretty good explanation of a way to distinguish between good (clean) and bad code. While almost every code produces some WTFs per second, clean code will make your colleagues less mad, software easy to manage and grow, and will enable the company to evolve.


To get to the state shown on the left side of the image above, developers should always strive to apply a set of principles, patterns and well known best practices.


### Code can be considered clean when it:

* is elegant, efficient and simple to read
* is self-explanatory and has straightforward logic without the need of explanatory comments
* favors exception throwing instead of error codes and has complete and centralized error handling
* reflects SOLID principles, especially the single responsibility principle
* does not contain code duplications across modules and multiple projects
* favors composition over inheritance (does not contain class explosion)
* utilizes design patterns
* is easy to test
* is well formatted
* follows well defined naming conventions and coding style is consistent
* has classes that tend to be small and methods that do not have long list of input parameters
* is well (and consistently) organized on the directory, project and solution level.


### Naming

Developers do write code for machines to execute it, but also for other developers to maintain it and extend it. Therefore, code should be easy to read and understand and should reflect shared vocabulary used by all team members involved in the project.

In general naming should be:

* well thought through, and should reflect business concepts and rules
* consistent throughout the code base
* truthful - this applies especially on the method level, where method name should not be too general or misleading due to method's side effects


### Comments

Comments can sometimes be viewed as a helpful part of the source code. 
However, developers should avoid writing them. The comments can be viewed as noise if they do not contain significant information. Even worse, if they aren't well maintained, they can lead developers to false conclusions. 

In general, if the code needs comments to clarify its purpose, it is a sign that it should be rewritten. The exceptions to this rule are:

* Describing an example
* Pointing to resources in documentation
* Todos
* Swagger documentation


### Methods

Methods should be short and have a single responsibility.
Logic contained in a single method should reflect the same level of abstraction. Mixing different levels of abstraction in the same function is a code smell.

* Methods should not have side effects, and method names should reflect exactly what they are doing.
* Prefer methods no longer than 10 lines.
* The number of input parameters should be up to 4. If there is a need for more parameters, consider creating a DTO.
* Methods should not have multiple return parameters (exception is TryDoSomething pattern which returns bool and resulting object via out param).
* Avoid using flag arguments. Split the method into several independent methods that can be called from the client without the flag.

### Code order

C# has no specific requirements for the code order inside a class. This is great for us because it gives us the freedom to place code wherever we want, but that doesn't mean we should just put it anywhere and call it a day.

As we mentioned before, the code we write must be understandable to developers as well as the machines. In this context, understandable code must tell a story about the class we are writing, just as if we were writing a newspaper article. First, you get the high-level information, and as you continue reading you dive into more details. Related code should be vertically close, and callers should be above the callees, if possible. Alongside these guidelines, we use the following order:

1. private fields
2. public properties
3. constructors
4. static methods
5. instance methods

### Abstraction & encapsulation

It is a good practice to expose abstract interfaces that allow its users to manipulate the essence of the data, without having to know its implementation.

When modeling entity classes, encapsulating data state details leads to increased control over entity access and manipulation, along with providing clean, well-defined ways to interact with entities. Simply put, details should be hidden and behavior exposed.


### Law of Demeter

> *Each unit must have limited knowledge of other units: it must see only units closely related to the current unit.*

In other words, the Law of Demeter principle states that a module (class) should not know about the inner details of the objects it manipulates.

![LoD](/resources/law-of-demeter.png)


```c#
    human
        .getDigestiveSystem() // 1. level of details
        .getStomach()         // 2. level of details
        .add(new Cake());    
```

The above code can be viewed as a *sausage code* and expresses a code smell:

* lack of encapsulation - Human class exposes too many details, making other users of this code dependent on low-level detail code
* lack of abstractions - if the eating behavior is changed, the caller code should also change
* hard to test
* hard to read

Instead, rewrite the code above to:

```c#
    human.Eat(new Cake()))
```



### Anemic model as anti-pattern

Software development that solves non-trivial problems and contains rich business logic produces much more complex code than simple CRUD-based software development. To model the business entities with integrity, their data states should be hidden while exposing the methods to interact with the entity.

Anemic models are known in the industry as business entities modeled as simple DTOs, leaving the purpose interpretation and interaction responsibilities to the calling code (usually services). Usually, the business logic ends up being implemented in service classes. This can lead to code duplications, leaking of the business logic into other layers, missing or corrupted entity validations, and many more issues.

Ways to avoid an anemic domain model are:

* Use private setters and expose methods to update data state.
* Always validate the state of entities - your entities must self-validate and not rely on API (contract) validation.
* Constructors without parameters are allowed to be only private as they are used by the ORM.
* Avoid primitive obsession.


#### Primitive obsession

Primitive types are basic built-in building blocks of a language. Examples are integers, dates, strings, etc. Primitive obsession is a programming style that heavily relies on primitives.

Designing business entities relying on primitive types can result in poor or decentralized entity state validation. It's often breaking the single responsibility principle.

Below is an example of a piece of code that avoids the primitive obsession:

```c#
    public class CompanyEvent : Entity
    {
        private readonly List<Member> _members = new();     // List of members is private and cannot be manipulated freely in calling code.
        public IReadOnlyList<Member> Members => _members;  //  Calling code has access to IReadOnlyList, so data integrity is protected.
        public Name Name { get; } = default!;             //   Name is not just a string, it is Name class that implements the validation rules.
        public TimeFrame Time { get; set; } = default!;   //   TimeFrame class is implementing the date validation rules.

        /// Here you define methods i.e. behaviors of your entity that needs to be exposed.
    }
```

### Don't Repeat Yourself (DRY)

The DRY principle promotes reusable component creation so that the code responsible for one thing appears only in one place. It suggests avoiding repetition by replacing duplicate logic or code snippets with shared methods or classes. That way, code is easier to maintain and update, and there is a smaller chance of errors and unwanted program behavior.


**Bad example:**

```c#
public class UserService
{
    public void CreateUser(UserDto userDto, string currentUserRole)
    {
        if (currentUserRole != "Admin")
        {
            throw new UnauthorizedAccessException("Only admins can create users.");
        }

        // Create user logic
        var user = new User(userDto.Name, userDto.Email);

        // Send notification
        var notification = new Notification
        {
            Recipient = user.Email,
            Subject = "Welcome!",
            Body = $"Hello {user.Name}, your account has been created."
        };
        EmailService.Send(notification);
    }
}

public class ProductService
{
    public void CreateProduct(ProductDto productDto, string currentUserRole)
    {
        if (currentUserRole != "Admin")
        {
            throw new UnauthorizedAccessException("Only admins can create products.");
        }

        // Create product logic
        var product = new Product(productDto.Name, productDto.Price);

        // Send notification
        var notification = new Notification
        {
            Recipient = "admin@company.com",
            Subject = "New Product Created",
            Body = $"Product {product.Name} was added to the catalog."
        };
        EmailService.Send(notification);
    }
}
```

Here are two clear violations:
- Role check logic is duplicated.
- Notification creation and sending is repeated with minor differences.

**Good example:** 

```c#
public class AuthorizationService
{
    public static void EnsureAdmin(string role)
    {
        if (role != "Admin")
        {
            throw new UnauthorizedAccessException("Admin privileges required.");
        }
    }
}

public class NotificationFactory
{
    public static Notification CreateUserWelcomeNotification(User user)
    {
        return new Notification
        {
            Recipient = user.Email,
            Subject = "Welcome!",
            Body = $"Hello {user.Name}, your account has been created."
        };
    }

    public static Notification CreateProductCreatedNotification(Product product)
    {
        return new Notification
        {
            Recipient = "admin@company.com",
            Subject = "New Product Created",
            Body = $"Product {product.Name} was added to the catalog."
        };
    }
}

public class UserService
{
    public void CreateUser(UserDto userDto, string currentUserRole)
    {
        AuthorizationService.EnsureAdmin(currentUserRole);

        var user = new User(userDto.Name, userDto.Email);

        var notification = NotificationFactory.CreateUserWelcomeNotification(user);
        EmailService.Send(notification);
    }
}

public class ProductService
{
    public void CreateProduct(ProductDto productDto, string currentUserRole)
    {
        AuthorizationService.EnsureAdmin(currentUserRole);

        var product = new Product(productDto.Name, productDto.Price);

        var notification = NotificationFactory.CreateProductCreatedNotification(product);
        EmailService.Send(notification);
    }
}
```

Now:
- Role-check logic lives in one place
- Notification creation is centralized and reusable
- Code is easier to read, maintain, and extend (e.g., log notifications, customize formatting, etc.)

Although the DRY principle is very useful, there *are* situations where applying the DRY principle might lead to greater complexity. The key point is that it's not an excuse to overcomplicate the code. If making something reusable introduces more complexity, it could be better to leave the code as it is.

Keeping the code "dry" isn't just about removing the redundant code. There are additional strategies that can help achieve it. SOLID principles and design patterns are an integral part of that pursuit.

### Keep It Simple, Stupid (KISS)

The KISS principle is a design philosophy emphasizing simplicity and avoiding unnecessary complexity. The idea is that most systems work best if they are kept simple rather than made complex. Simplicity should be a key goal in design, and unnecessary complexity should be avoided.

The following are some detailed aspects and examples to illustrate the KISS principle.

#### 1. Simple Code Structure

Avoid complex and nested structures when simpler ones can do the job.

**Bad Example** (using complex structure)**:**

```c#
if (condition1)
{
    if (hasPermission)
    {
        Console.WriteLine("You can access the content.");
    }
    else
    {
        Console.WriteLine("You need permission to access the content.");
    }
}
else
{
    Console.WriteLine("You must meet condition1 to access the content.");
}
```

**Good Example** (using simple structure)**:**

```c#
if (!condition1)
{
    Console.WriteLine("You must meet condition1 to access the content.");
}
else if (!hasPermission)
{
    Console.WriteLine("You need permission to access the content.");
}
else
{
    Console.WriteLine("You can access the content.");
}
```

#### 2. Use Clear and Descriptive Names

Use meaningful variable and method names to make the code self-explanatory.

**Bad Example** (using vague variable names (e.g. `a`, `b` and `result`)):

```c#
int a = 10;
int b = 20;
int result = a + b;
```

**Good Example** (using clear and descriptive variable names):

```c#
int numberOfApples = 10;
int numberOfOranges = 20;
int totalFruits = numberOfApples + numberOfOranges;
```

#### 3. Simplify Logic with Smaller Methods

Instead of writing a long method with many responsibilities, break it into smaller ones.

**Bad Example** (a long method with complex logic) **:**

```c#
public void ProcessOrder(Order order)
{
    if (order != null)
    {
        // Validate order
        if (order.IsValid)
        {
            // Process payment
            if (order.PaymentMethod == "CreditCard")
            {
                // Process credit card payment
            }
            else if (order.PaymentMethod == "PayPal")
            {
                // Process PayPal payment
            }
        }
    }
}
```

**Good Example** (short methods with single responsibilities) **:**

```c#
public void ProcessOrder(Order order)
{
    if (order == null || !order.IsValid) return;

    ProcessPayment(order);
}

private void ProcessPayment(Order order)
{
    switch (order.PaymentMethod)
    {
        case "CreditCard":
            // Process credit card payment
            break;
        case "PayPal":
            // Process PayPal payment
            break;
    }
}
```

#### 4. Avoid Over-Engineering

Design solutions to meet the current requirements and align with the program's development direction rather than hypothetical future needs.

**Bad Example** (multiple interfaces and classes manage configurations, which can be unnecessarily complex for a simple application) **:**

```c#
public interface IConfigurationProvider
{
    string GetSetting(string key);
}

public class FileConfigurationProvider : IConfigurationProvider
{
    private readonly string _filePath;

    public FileConfigurationProvider(string filePath)
    {
        _filePath = filePath;
    }

    public string GetSetting(string key)
    {
        return File.ReadAllText(_filePath);
    }
}

public class EnvironmentConfigurationProvider : IConfigurationProvider
{
    public string GetSetting(string key)
    {
        return Environment.GetEnvironmentVariable(key);
    }
}

public class ConfigurationManager
{
    private readonly IConfigurationProvider _provider;

    public ConfigurationManager(IConfigurationProvider provider)
    {
        _provider = provider;
    }

    public string GetConfiguration(string key)
    {
        return _provider.GetSetting(key);
    }
}

// Usage
var configManager = new ConfigurationManager(new FileConfigurationProvider("config.txt"));
string setting = configManager.GetConfiguration("SomeKey");
```

**Good Example:** Using a single class that reads from a configuration file or environment variables directly.

```c#
public class ConfigurationManager
{
    private readonly string _filePath;

    public ConfigurationManager(string filePath)
    {
        _filePath = filePath;
    }

    public string GetSetting(string key)
    {
        string value = Environment.GetEnvironmentVariable(key);
        if (!string.IsNullOrEmpty(value))
        {
            return value;
        }

        var settings = GetSettingsFromFile();

        return settings.ContainsKey(key) ? settings[key] : null;
    }

    public Dictionary<string, string> GetSettingsFromFile()
    {
        return File.ReadAllLines(_filePath)
            .Select(line => line.Split('='))
            .ToDictionary(parts => parts[0], parts => parts[1]);
    }
}

// Usage
var configManager = new ConfigurationManager("config.txt");
string setting = configManager.GetSetting("SomeKey");
```

#### 5. Use Built-In Libraries

Leverage existing libraries and frameworks to avoid reinventing the wheel. For example, instead of implementing your sorting algorithm, use the built-in methods provided by the .NET framework.

NOTE: If a 3rd party library/framework solves the issue, always check its reliability, history, and development direction and make sure its author(s) are trustworthy before applying it to the project. Also, investigate how to replace it in case of compatibility or compliance issues.

#### 6. Reduce Redundancy

Avoid duplicate code by reusing methods and classes, and applying DRY.

**Bad Example** (printing the message for every type of user separately) **:**

```c#
public void PrintWelcomeMessage(string userType)
{
    if (userType == "Admin")
    {
        Console.WriteLine("Welcome, Admin!");
    }
    else if (userType == "User")
    {
        Console.WriteLine("Welcome, User!");
    }
}
```

**Good Example** (implementing generic method for printing message) **:**

```c#
public void PrintWelcomeMessage(string userType)
{
    Console.WriteLine($"Welcome, {userType}!");
}
```

### YAGNI (You Aren’t Gonna Need It)

> *Don't write code for features you think you might need in the future — only write what you need right now.*

YAGNI protects developers from the trap of over-engineering, which adds complexity, increases maintenance overhead, and often results in features that are never used. It encourages lean, focused development with simpler code, faster delivery, and easier debugging.

**Bad Example** (only sending e-mails is required) **:**

```c#
public interface INotificationService
{
    void SendEmail(string to, string subject, string body);
    void SendSms(string number, string message);
    void SendPushNotification(string deviceToken, string message);
}

public class NotificationService : INotificationService
{
    public void SendEmail(string to, string subject, string body)
    {
        Console.WriteLine("Sending Email...");
    }

    public void SendSms(string number, string message)
    {
        // Not needed now
        throw new NotImplementedException();
    }

    public void SendPushNotification(string deviceToken, string message)
    {
        // Not needed now
        throw new NotImplementedException();
    }
}
```

In this case, the system only needs email notifications, but the developer anticipates future needs for SMS and push notifications. This adds dead code, untested methods, and unnecessary complexity.

**Good example:**

```c#
public class EmailNotificationService
{
    public void SendEmail(string to, string subject, string body)
    {
        Console.WriteLine("Sending Email...");
    }
}
```

This version delivers exactly what's required right now — email notifications only. If requirements change in the future, the code can be extended when it's needed, based on real use cases.

#### When to Break YAGNI?

There are rare cases where you might consider building ahead — for example:

* When the cost of refactoring later is prohibitively high
* When future requirements are certain (e.g., clearly scoped roadmap)
* When extending architecture is nearly free and adds no real complexity

Even then, such decisions should be intentional and justified — not based on “just in case” thinking.