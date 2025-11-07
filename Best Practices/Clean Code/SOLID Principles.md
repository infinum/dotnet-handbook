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

**Bad example** 

```c#
public interface ISmartDevice
{
    void TurnOn();
    void TurnOff();
    void SetTemperature(int temperature);
}

public class SmartLight : ISmartDevice
{
    public void TurnOn()
    {
        // Turn light on
    }

    public void TurnOff()
    {
        // Turn light off
    }

    public void SetTemperature(int temperature)
    {
        throw new NotSupportedException("Lights do not support temperature settings.");
    }
}

public class SmartThermostat : ISmartDevice
{
    public void TurnOn()
    {
        // Turn thermostat on
    }

    public void TurnOff()
    {
        // Turn thermostat off
    }

    public void SetTemperature(int temperature)
    {
        // Set temperature logic
    }
}
```

Here, SmartLight is forced to implement `SetTemperature`, which doesn't make any sense for a light. That's the violation of ISP.

**Good example** (using smaller, more specific interfaces to avoid unnecessary dependencies)**:**

```c#
public interface IDevice
{
    void TurnOn();
    void TurnOff();
}

public interface ITemperatureDevice
{
    void SetTemperature(int temperature);
}

public class SmartLight : IDevice
{
    public void TurnOn()
    {
        // Turn light on
    }

    public void TurnOff()
    {
        // Turn light off
    }
}

public class SmartThermostat : IDevice, ITemperatureDevice
{
    public void TurnOn()
    {
        // Turn thermostat on
    }

    public void TurnOff()
    {
        // Turn thermostat off
    }

    public void SetTemperature(int temperature)
    {
        // Set temperature logic
    }
}
```

Now, each class only implements the interfaces that make sense for its role. SmartLight is not burdened with irrelevant methods.

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
