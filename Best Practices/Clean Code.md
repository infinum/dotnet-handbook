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
        .add(new Cake()))    
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

### SOLID

#### Single Responsibility Principle (SRP):

>  *A class should have only one reason to change.*

In other words, a class/method should have only one job (responsibility). When we say a class should have "single responsibility," it doesn't necessarily mean the class can only do one thing. Instead, it implies that the class should have a single, well-defined purpose or responsibility within the system.

**Bad example** (mixing multiple responsibilities within the same class)**:**

```c#
public class CustomerService
{
    public void AddCustomer(Customer customer)
    {
        // validation logic (or some other business logic)

        // Data access logic for adding a customer
    }
}
```

**Good example** (separating classes that handle different responsibilities)**:**

```c#
public class CustomerService
{
    private readonly ICustomerRepository _customerRepository;

    public CustomerService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }

    public void AddCustomer(Customer customer)
    {
        var validationResult = new CustomerValidator().Validate(customer);

        if (!validationResult.IsValid)
        {
            // handle errors
        }

        _customerRepository.Add(customer);
    }
}

public interface ICustomerRepository
{
    void Add(Customer customer);
}

public class CustomerRepository : ICustomerRepository
{
    public void Add(Customer customer)
    {
        // Data access logic for adding a customer
    }
}

public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        // validation logic
    }
}
```

In the example above, the `AddCustomer` method encapsulates the responsibility of adding a customer. It validates data in the `CustomerValidator` and delegates the actual data storage task to the `ICustomerRepository`, making an appropriate separation of concerns. The repository handles the business logic specific to adding a customer, the validator handles the validation, and the repository handles the data access logic.

If the way customers are added needs modification (e.g., validation rules change), only the `CustomerValidator` needs to be updated, thus adhering to the SRP.

#### Open-closed Principle (OCP)

>  *Software entities (such as classes, modules, and functions) should be open for extension but closed for modification.* 

This means you should be able to add new functionality without changing existing, tested code. Achieving this often involves polymorphism, abstraction, and coding to interfaces, rather than relying solely on class inheritance.

**Bad example:**

```c#
public class SavingAccount
{
    public decimal CalculateInterest(AccountType accountType)
    {
        if(AccountType=="Regular")
        {
            //Calculate interest for Regular account type 
        }
        else if(AccountType=="Salary")
        {
            //Calculate interest for Salary account type 
        }
    }
}
```

The `SavingAccount` class violates the OCP by containing a single method that calculates interest based on the account type passed as a parameter. If a new type of saving account is added, the `SavingAccount` class needs to be modified to add conditional logic for the new account type. This violates the OCP as the class is open for modification, which can lead to potential issues and bugs.

**Good example:**

```c#
public interface ISavingAccount
{
   decimal CalculateInterest();
}

public class RegularSavingAccount : ISavingAccount
{
  public decimal CalculateInterest()
  {
    // Calculate interest for regular account type 
  }
}

public class SalarySavingAccount : ISavingAccount
{
  public decimal CalculateInterest()
  {
    //Calculate interest for Salary account type 
  }
}
```

The `ISavingAccount` interface defines a contract for calculating interest on saving accounts. The `RegularSavingAccount` and `SalarySavingAccount` classes implement this interface and provide specific implementations for calculating interest based on account type. When adding a new saving account type in the future, a new class that implements `ISavingAccount` can be created without modifying the existing ones. This way, the open-closed principle is satisfied as the classes are open for extension (new implementations can be added) but closed for modification (existing implementations remain unchanged).

#### Liskov Substitution Principle (LSP)

>  *Let q(x) be a property provable for any object x of type T. Then q(y) should be provable for any object y of type S where S is a subtype of T.*

Every subclass or derived class should be substitutable for their base or parent class. In simpler terms, if S is a subtype of T, then objects of type S may replace objects of type T without altering the correctness of the program.

**Bad example** (might involve one subclass behaving differently than another in a way that violates the contract defined by the superclass)**:**

```c#
public interface IBird
{
    string Fly();
}

public class Sparrow : IBird
{
    public void Fly()
    {
        Console.WriteLine("Sparrow flying.");
    }
}

public class Ostrich : IBird
{
    public override void Fly()
    {
        throw new NotSupportedException("Ostriches can't fly!");
    }
}

class Program
{
    static void Main(string[] args)
    {
        IBird bird = new Sparrow();
        bird.Fly(); // Sparrow flying.

        bird = new Ostrich();
        bird.Fly(); // Throws NotSupportedException
    }
}
```

Both `Sparrow` and `Ostrich` classes implement the `IBird` interface. However, the first implements the `Fly()` method while the second throws the NotSupported exception. This means using the `Ostrich` type as a replacement for `IBird` leads to errors in the code execution and deviates from the expected program behavior (program completes successfully).

**Good example:**

```c#
public interface IBird
{
    void Eat();
}

public interface IFlyingBird : IBird
{
    void Fly();
}

public class Sparrow : IFlyingBird
{
    public void Eat()
    {
        Console.WriteLine("Sparrow eating seeds.");
    }

    public void Fly()
    {
        Console.WriteLine("Sparrow flying.");
    }
}

public class Ostrich : IBird
{
    public void Eat()
    {
        Console.WriteLine("Ostrich eating plants.");
    }

    // No Fly method here - LSP respected
}

class Program
{
    static void Main(string[] args)
    {
        IBird bird = new Sparrow();
        bird.Eat();

        bird = new Ostrich();
        bird.Eat();
    }
}
```

The output is:

```
Sparrow eating seeds.
Ostrich eating plants.
```

In the `Main` method, `Sparrow` and `Ostrich` instances are created and assigned to a variable of type `IBird`. This behavior demonstrates substitutability, where objects of the subclasses (`Sparrow` and `Ostrich`) can be used interchangeably with superclass objects (`IBird`) without affecting the program correctness.

#### Interface Segregation Principle (ISP)

>  *A client shouldn't ever need to depend on methods it does not use.*

Interfaces should be as small as possible and should not contain methods that are not closely related to each other. Otherwise, splitting one interface into multiple smaller ones should be considered.

**Bad example** (forcing clients to depend on the unnecessary method `FindById` when only `Add` is needed)**:**

```c#
public class Customer
{
    public Guid Id { get; }
    public string Name { get; set; }

    protected Customer()
    {
    }

    public Customer(string name, Guid? id = default)
    {
        if (id != default)
        {
            Id = id;
        }

        Name = name;
    }
}

public interface ICustomerService
{
    void Add(Customer customer);

    Customer FindById(int id);
}

public class CustomerService : ICustomerService
{
    public void Add(Customer customer)
    {
        // Add logic
    }

    public Customer FindById(int id)
    {
        // Find logic
        return new Customer("name", id);
    }
}

public class CompanyService
{
    private readonly ICustomerService _customerService;

    public CompanyService(ICustomerService customerService)
    {
        _customerService = customerService;
    }

    public void Add(string Name, Customer[] customers)
    {
        // Company creation logic
        foreach (var customer in customers)
        {
            _customerService.Add(customer);
        }
    }
}
```

**Good example** (using smaller, more specific interfaces to avoid unnecessary dependencies)**:**

```c#
public class Customer
{
    public Guid Id { get; }
    public string Name { get; set; }

    protected Customer()
    {
    }

    public Customer(string name, Guid? id = default)
    {
        if (id != default)
        {
            Id = id;
        }

        Name = name;
    }
}

public interface IAddCustomerService
{
    void Add(Customer customer);
}

public interface IFindCustomerService
{
    Customer FindById(int id);
}

public class CustomerService : IAddCustomerService, IFindCustomerService
{
    public void Add(Customer customer)
    {
        // Add logic
    }

    public Customer FindById(int id)
    {
        // Find logic
        return new Customer("name", id);
    }
}

public class CompanyService
{
    private readonly IAddCustomerService _addCustomerService;

    public CompanyService(IAddCustomerService addCustomerService)
    {
        _addCustomerService = addCustomerService;
    }

    public void Add(string Name, Customer[] customers)
    {
        // Company creation logic
        foreach (var customer in customers)
        {
            _addCustomerService.Add(customer);
        }
    }
}
```

#### Dependency Inversion Principle (DIP)

>  *A. High-level modules should not import anything from low-level modules. Both should depend on abstractions (e.g., interfaces).* 
>  *B. Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.*

**Bad example** (depending on concrete implementations, tightly coupling high-level and low-level modules)**:**

```c#
public class CustomerService
{
    private readonly SqlCustomerRepository _customerRepository;

    public CustomerService()
    {
        _customerRepository = new SqlCustomerRepository();
    }

    public void AddCustomer(Customer customer)
    {
        _customerRepository.Add(customer);
    }
}

public class SqlCustomerRepository
{
    public void Add(Customer customer)
    {
        // SQL-specific logic for adding a customer
    }
}
```

* The `CustomerService` class directly depends on the `SqlCustomerRepository` class, creating a tight coupling between the high-level `CustomerService` and the low-level `SqlCustomerRepository`. If data storage needs to be changed (to a different database or storage system), `CustomerService` must be modified.
* Any changes in `SqlCustomerRepository` can impact the `CustomerService` class, leading to increased maintenance overhead and potential bugs.
* The code above violates the Open-closed principle because the class is not closed for modification.
* Testing `CustomerService` in isolation is more difficult because `SqlCustomerRepository` can't be mocked without using advanced techniques like dependency injection frameworks or reflection.

**Good example** (depending on abstractions to decouple high-level and low-level modules)**:**

```c#
public class CustomerService
{
    private readonly ICustomerRepository _customerRepository;

    public CustomerService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }

    public void AddCustomer(Customer customer)
    {
        _customerRepository.Add(customer);
    }
}

public interface ICustomerRepository
{
    void Add(Customer customer);
}

public class SqlCustomerRepository : ICustomerRepository
{
    public void Add(Customer customer)
    {
        // SQL-specific logic for adding a customer
    }
}
```

The `CustomerService` class depends on an abstraction (`ICustomerRepository`) rather than a concrete class. The dependency is injected into the `CustomerService` constructor, promoting loose coupling and adherence to the DIP. The benefits of this approach are the following:
* Decoupling the high-level `CustomerService` from the low-level `SqlCustomerRepository` so it can work with any implementation of `ICustomerRepository`.
* Simplification of data storage switching (e.g., from SQL to a NoSQL database) without modifying the `CustomerService` class.
* Easier writing of unit tests for `CustomerService` because the `ICustomerRepository` interface can be mocked, allowing `CustomerService` to be tested in isolation from the data layer.

### Don't Repeat Yourself (DRY)

The DRY principle promotes reusable component creation so that the code responsible for one thing appears only in one place. It suggests avoiding repetition by replacing duplicate logic or code snippets with shared methods or classes. That way, code is easier to maintain and update, and there is a smaller chance of errors and unwanted program behavior.

**Good example:** 

```c#
public interface IShape
{
    public double CalculateArea();
}

public class Rectangle : IShape
{
    public double CalculateArea()
    {
        // Calculation logic for rectangle's area
    }
}

public class Circle : IShape
{
    public double CalculateArea()
    {
        // Calculation logic for circle's area
    }
}
```

We've created a method `Area` in an interface `IShape`, so there is no code duplication for calculating areas in each subclass (`Rectangle` and `Circle`). Instead, each subclass only needs to implement the logic specific to its shape.

**Bad example** (repeating code for area calculation)**:**

```c#
public interface IShape
{
    public double CalculateArea();
}

public class Rectangle : IShape
{
    public double CalculateArea()
    {
        // Calculation logic for rectangle's area
    }
}

public class Circle : IShape
{
    public double CalculateArea()
    {
        // Calculation logic for circle's area
    }
}

public class AreaCalculator
{
    public double CalculateRectangleArea(double length, double width)
    {
        return length * width;
    }

    public double CalculateCircleArea(double radius)
    {
        return Math.PI * radius * radius;
    }
}
```

In the code above, the code for calculating area is implemented in each class implementing `IShape` and duplicated in `AreaCalculator`. In the latter, there's a redundancy in the method signatures as the equivalent logical operation is split into different methods.

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