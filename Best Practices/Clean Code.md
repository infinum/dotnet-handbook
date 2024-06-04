### What is the clean code ?


![celanCodeImg](/resources/wtf.png)


Image above is a pretty good explanation on how to distinguish between good and bad code. It is WTF/s, and almost every code will have it, but good code will make your colleagues less mead, software easy to manage and grow, and enable the company to evolve.  


To develop code that will put us on the left side of the above image, developers should always strive to apply a set of principles, patterns and well known best practices.


### Clean code reflects in :
* Elegant, efficient and simple to read code.
* It is self-explanatory, logic is straightforward, without the need of explanatory comments.
* If favors exception throwing instead of error codes, and has complete and centralized error handling.
* It should reflect SOLID principles, especially single responsibility.
* It does not contain code duplications across modules and multiple projects.
* It favors composition over inheritance (does not contain class explosion) and utilizes design patterns.
* It is easy to test.
* It is well formatted.
* It follows well defined naming conventions and coding style is consistent.
* Classes tend to be small and methods does not have long list of input parameters.
* It is well (and consistiently) organized on the directory, project and solution level.


### Naming

Developers write code for machines to execute it, but for other developers to maintain it and extend it. So code should be easy to read and understand.
Code should reflect shared vocabulary used by all team members involved in the project.

So in general naming should be:

* Well thought through, and should reflect business concepts and rules.
* Consistent through the software.
* Truthful - this applies especially on the method level, where method name should not be too general or misleading due to method's side effects.


### Comments

Comments are part of source code, and if not consisted of significant info, then comments act as noise, and even worse if not well maintained they can lead developers to false conclusions. In general, developers should avoid writing the comments. If the code is unclear, it is a sign it should be rewritten.

Exceptions:

* Describing an example
* Pointing to resources in documentation
* Todos
* Swagger documentation


### Methods

Methods should be short and have single responsibility.
Logic contained in a single method should reflect the same level of abstraction. Mixing different levels of abstraction in the same function is a code smell.

* Methods should not have side effects, and method names should reflect exactly what they are doing.
* Prefer methods no longer than 10 lines.
* Number of input parameters should be up to 4. If there is a need for more params, consider creating a DTO.
* Methods should not have multiple return params (exception is TryDoSomething pattern which returns bool and resulting object via out param).
* Avoid using flag arguments. Split method into several independent methods that can be called from the client without the flag.


### Abstraction & Encapsulation

It is a good practice to expose abstract interfaces that allow its users to manipulate the essence of the data, without having to know its implementation.

When modeling entity classes, encapsulating data state details, lead to increased control over entity access and manipulation, providing a clean, well defined ways to interact with entities. Simply put, you should hide details, and expose behavior.


### Law of Demeter

> *Each unit must have limited knowledge of other units: it must see only units closely related to the current unit.*

In other words, the Law of Demeter principle states that a module (class) should not have the knowledge on the inner details of the objects it manipulates.

![LoD](/resources/law-of-demeter.png)


```c#
    human
        .getDigestiveSystem() // 1. level of details
        .getStomach()         // 2. level of details
        .add(new Cake()))    
```

Above code can be described as *sausage code* and express obvious code smell:

* lack of encapsulation - human class exposes too much details, making other users of this code dependent on low level detail code.
* lack of abstractions - if the eat behavior is changed, caller code should also change.
* hard to test
* hard to read

Instead above code should be rewritten to :

```c#
    human.Eat(new Cake()))
```


### Anemic model as anti-pattern

In development of software that solves non-trivial problems and contains rich business logic, code is much more complex than in simple CRUD-based software. To model the busines entities with integrity, their data states should be hidden while exposing the methods to interact with entity.

Anemic models are known in industry as business entities modeled as simple DTOs, leaving the purpose interpretation and interaction responsibilities to the calling code (usually services). Usually the business logic ends up being implemented in service classes, which can lead to code duplications, leaking of the business logic into other layers, missing or corrupted entity validations, and many more issues.

Ways to avoid an anemic domain model:

* Use private setters and expose methods to update data state.
* Always validate the state of entities - your entities must self-validate and not relay on API (contract) validation.
* Constructors without parameters are allowed to be only private as they are used by the ORM.
* Avoid primitive obsession.


### Primitive obsession

Primitive fields are basic built-in building blocks of a language such as int, dates, strings, etc. Primitive Obsession is a programming style that heavily relies on primitives.
Designing business entities relying on primitive types can result in poor or decentralized entity state validation, and is often breaking the single responsibility principle.

```c#
    public class CompanyEvent : Entity
    {
        private readonly List<Member> _members = new();     // list of members is private and can not be manipulated freely in calling code.
        public IReadOnlyList<Member> Members => _members;  //  calling code have access to IReadOnlyList, so data integrity is protected.
        public Name Name { get; } = default!;             //   Name is not just a string, it is Name class that implements validation rules
        public TimeFrame Time { get; set; } = default!;   //   TimeFrame class is implementing date validation rules

        /// here you define methods i.e. behaviors of your entity you need to expose
    }
```

### SOLID

#### Single Responsibility Principle (SRP):

A class/method should have one and only one reason to change, meaning that a class/method should have only one job (responsibility). 
When we say a class should have "single responsibility," it doesn't necessarily mean that the class can only do one thing. Instead, it implies that the class should have a single, well-defined purpose or responsibility within the system.

**Good example:** Separate classes handle different responsibilities

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

In the provided example, the `AddCustomer` method encapsulates the responsibility of adding a customer. It validates data in `CustomerValidator` and delegates the actual data storage task to the `_customerRepository`, which is an appropriate separation of concerns. This method handles the business logic specific to adding a customer, while validator handles the validation and the data access logic is handled by the repository.
If the way customers are added needs to be modified, e.g. validation rules change, you would only need to update `CustomerValidator`, thus adhering to the SRP.

**Bad example:** Mixing multiple responsibilities within the same class.

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

#### Open Closed Principle (OCP)

Objects or entities should be open for extension but closed for modification. Class inheritance is not always the best way, coding to an interface is an integral part of SOLID.

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

The `ISavingAccount` interface defines a contract for calculating interest on saving accounts. The `RegularSavingAccount` and `SalarySavingAccount` classes implement this interface and provide specific implementations for calculating interest based on account type. If a new type of saving account is introduced in the future, you can create a new class that implements `ISavingAccount` without modifying the existing classes. This follows the Open/Closed Principle as the classes are open for extension (you can add new implementations) but closed for modification (existing implementations remain unchanged).

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

The `SavingAccount` class violates the OCP by containing a single method that calculates interest based on the account type passed as a parameter. If a new type of saving account is introduced, you need to modify the `SavingAccount` class to add conditional logic for the new account type. This violates the OCP as the class is open for modification, which can lead to potential issues and bugs.

### Liskov Substitution Principle (LSP)

>  *Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T.*

This means that every subclass or derived class should be substitutable for their base or parent class.  In simpler terms, if S is a subtype of T, then objects of type T may be replaced with objects of type S without altering any of the desirable properties of the program.

**Good example:**

```c#
public interface IFruit
{
    string GetColor();
}

public class Apple : IFruit
{
    public string GetColor()
    {
        return "Red";
    }
}

public class Orange : IFruit
{
    public string GetColor()
    {
        return "Orange";
    }
}

class Program
{
    static void Main(string[] args)
    {
        IFruit fruit = new Orange();
        Console.WriteLine($"Color of Orange: {fruit.GetColor()}");

        fruit = new Apple();
        Console.WriteLine($"Color of Apple: {fruit.GetColor()}");
    }
}
```

The output is:

```
Color of Orange: Orange
Color of Apple: Red
```

This is expected behaviour, in `Main` method we create instances of `Apple` and `Orange` and assign them to a variable of type `IFruit`. This demonstrates substitutability, where objects of the subclasses (`Apple` and `Orange`) can be used interchangeably with objects of the superclass (`IFruit`) without affecting the behavior of the program.

**Bad example:** might involve one subclass behaving differently than another in a way that violates the contract defined by the superclass.

```c#
public interface IFruit
{
    string GetColor();
}

public class Apple : IFruit
{
    public string GetColor()
    {
        return "Red";
    }

    public bool IsTasty()
    {
        return true;
    }
}

public class Orange : IFruit
{
    public string GetColor()
    {
        return "Orange";
    }

    public bool IsTasty()
    {
        return false;
    }
}

class Program
{
    static void Main(string[] args)
    {
        IFruit fruit = new Orange();
        Console.WriteLine($"Color of Orange: {fruit.GetColor()}");
        Console.WriteLine($"Is Orange tasty? {((Orange)fruit).IsTasty()}"); // runtime error
    }
}
```

The `Apple` and `Orange` classes both implement `IFruit`, but they introduce an additional method `IsTasty()` that's not defined in the `IFruit` interface. While `Apple` and `Orange` are still technically substitutable for `IFruit`, the introduction of `IsTasty()` in one subclass but not the other violates the LSP. Code expecting an `IFruit` may rely on `GetColor()` but not `IsTasty()`, leading to unexpected behavior if an `Orange` instance is substituted for an `Apple` instance or vice versa.

### Interface Segregation Principle (ISP)

A client should never be forced to implement an interface that it doesn't use, or clients shouldn't be forced to depend on methods they do not use.
This means interfaces should be small, and should not contain methods not critically linked. If that is the case, consider splitting one interface into multiple smaller interfaces.

**Good example:** Using smaller, more specific interfaces to avoid unnecessary dependencies.

```c#
public interface IAddable
{
    void Add();
}

public interface IFindable
{
    void FindById(int id);
}

public class CustomerService : IAddable, IFindable
{
    public void Add()
    {
        // Add logic
    }

    public void FindById(int id)
    {
        // Find logic
    }
}
```

**Bad example:** Forcing clients to depend on unnecessary methods when we only need `Add` and `FindById`.

```c#
public interface IService
{
    void Add();
    void Update();
    void Delete();
    void FindById(int id);
}

public class CustomerService : IService
{
    public void Add()
    {
        // Add logic
    }

    public void Update()
    {
        // Update logic
    }

    public void Delete()
    {
        // Delete logic
    }

    public void FindById(int id)
    {
        // Find logic
    }
}
```

### Dependency Inversion Principle (DIP)

Entities must depend on abstractions, not on concretions. The high-level modules must not depend on the low-level modules, but they both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions


**Good example:** Depending on abstractions to decouple high-level and low-level modules.

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

The `CustomerService` class depends on an abstraction (`ICustomerRepository`) rather than a concrete class. This decouples the high-level `CustomerService` from the low-level `SqlCustomerRepository`. By depending on an interface, `CustomerService` can work with any implementation of `ICustomerRepository`. This makes it easy to switch data storage mechanisms (e.g., switching from SQL to a NoSQL database) without modifying the `CustomerService` class. It's easier to write unit tests for `CustomerService` because you can mock the `ICustomerRepository` interface. This allows for testing `CustomerService` in isolation from the data layer. The dependency (`ICustomerRepository`) is injected into the `CustomerService` constructor, promoting loose coupling and adherence to the DIP.

**Bad example:** Depending on concrete implementations, tightly coupling high-level and low-level modules.

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
The `CustomerService` class directly depends on the `SqlCustomerRepository` class, creating a tight coupling between the high-level `CustomerService` and the low-level `SqlCustomerRepository`. If you need to change the data storage mechanism (e.g., to use a different database or storage system), you must modify the `CustomerService` class. This also violates the OCP because the class is not closed for modification. Testing `CustomerService` in isolation is more difficult because you can't easily mock the `SqlCustomerRepository` without using advanced techniques like dependency injection frameworks or reflection. Any changes in the `SqlCustomerRepository` can potentially impact the `CustomerService` class, leading to increased maintenance overhead and potential bugs.

### Don't Repeat Yourself (DRY)

DRY principle aims to avoid repetition by replacing duplicate logic or code snippets with shared methods or classes. It promotes creating reusable components so that future changes need to be implemented in only one place.

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

We’ve created a single method `Area` in an interface `IShape`. This way you avoid duplicating code for calculating areas in each subclass (`Rectangle` and `Circle`). Instead, each subclass only needs to implement the logic specific to its shape.

**Bad example:** Repeating code for area calculation

```c#
public class Shape
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

In the above code, there’s a clear redundancy in the method signatures, as the equivalent logical operation of calculating the area is divided amongst different methods.

It's important to understand the core of DRY principle. Picture this, you just finished a marathon coding session and the boss asks for one more feature before you call it a day. Your mind is exhausted but hey, you’re a fighter. Suddenly, your eyes fall on these mystery lines of code. You rack your brains, trying to decipher what in the world it’s doing. And then it strikes you, you’ve seen this before, somewhere else in the application. Familiar scenario? Welcome to the non-DRY world!

But worry not, the superhero of our story, the DRY principle, is here to the rescue! While the DRY principle promotes elimination of redundancy, its superpower lies in enhancing the readability, maintainability, testability and scalability of code.

Take it as cleaning your room. If everything’s in its place, finding that elusive comic book or gaming console becomes a breeze. Same goes with the code. Less clutter makes it easier to navigate and understand, and the fact that you’ve written less code means there’s less chance for bugs to creep in. I mean, who wouldn’t want that?

DRY principle is great, but there *are* situations where applying the DRY principle might lead to greater complexity. The key point to remember is, the DRY principle is not an excuse to overcomplicate your code. If making something reusable introduces more complexity, it could be better to leave the code as it is.

Keeping your code DRY isn’t just about removing the redundant code, there are additional strategies that can help you achieve it and SOLID principles and design patterns (e.g. Factory and Decorator patterns) are an integral part of this pursuit.
