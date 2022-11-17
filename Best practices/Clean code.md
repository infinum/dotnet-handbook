### What is the clean code ?


![celanCodeImg](/resources/wtf.png)


The image above is a pretty good explanation of how to distinguish between good and bad code. It is WTF/s, and almost every code will have it, but good code will make your colleagues less mead, software easy to manage and grow, and enable the company to evolve.  


To develop code that will put us on the left side of the above image, developers should always strive to apply a set of principles, patterns and well-known best practices.


### Clean code reflects in :
* Elegant, efficient and simple to read code.
* It is self-explanatory, and logic is straightforward, without the need for explanatory comments.
* It favors exception throwing instead of error codes, and has complete and centralized error handling.
* It should reflect SOLID principles, especially single responsibility.
* It does not contain code duplications across modules and multiple projects.
* It favors composition over inheritance (does not contain class explosion) and utilizes design patterns.
* It is easy to test.
* It is well formatted.
* It follows well defined naming conventions and the coding style is consistent.
* Classes tend to be small and methods do not have a long list of input parameters.
* It is well (and consistently) organized on the directory, project and solution level.


### Naming

Developers write code for machines to execute it, but for other developers to maintain and extend it. So code should be easy to read and understand.
The code should reflect the shared vocabulary used by all team members involved in the project.

So in general naming should be:

* Well thought through, and should reflect business concepts and rules.
* Consistent through the software.
* Truthful - this applies especially on the method level, where the method name should not be too general or misleading due to the method's side effects.


### Comments

Comments are part of source code, and if not consisted of significant info, then comments act as noise, and even worse if not well maintained they can lead developers to false conclusions. In general, developers should avoid writing comments. If the code is unclear, it is a sign it should be rewritten.

Exceptions:

* Describing an example
* Pointing to resources in the documentation
* Todos
* Swagger documentation


### Methods

Methods should be short and have a single responsibility.
The logic contained in a single method should reflect the same level of abstraction. Mixing different levels of abstraction in the same function is a code smell.

* Methods should not have side effects, and method names should reflect exactly what they are doing.
* Prefer methods no longer than 10 lines.
* The number of input parameters should be up to 4. If there is a need for more parameters, consider creating a DTO.
* Methods should not have multiple return params (exception is TryDoSomething pattern which returns bool and resulting object via out param).
* Avoid using flag arguments. Split the method into several independent methods that can be called from the client without the flag.


### Abstraction & Encapsulation

It is a good practice to expose abstract interfaces that allow its users to manipulate the essence of the data, without having to know its implementation.

When modeling entity classes, encapsulating data state details, lead to increased control over entity access and manipulation, providing clean, well defined ways to interact with entities. Simply put, you should hide details, and expose behavior.


### Law of Demeter

> *Each unit must have limited knowledge of other units: it must see only units closely related to the current unit.*

In other words, the Law of Demeter principle states that a module (class) should not know about the inner details of the objects it manipulates.

![LoD](/resources/law-of-demeter.png)


```c#
    human
        .getDigestiveSystem() // 1. level of details
        .getStomach()         // 2. level of details
        .add(new Cake())    
```

The above code can be described as *sausage code* and express an obvious code smell:

* lack of encapsulation - human class exposes too many details, making other users of this code dependent on low level detail code.
* lack of abstractions - if the eat behavior is changed, the caller code should also change.
* hard to test
* hard to read

Instead above code should be rewritten to :

```c#
    human.Eat(new Cake())
```


### Anemic model as an anti-pattern

In the development of software that solves non-trivial problems and contains rich business logic, code is much more complex than in simple CRUD-based software. To model the business entities with integrity, their data states should be hidden while exposing the methods to interact with the entity.

Anemic models are known in the industry as business entities modeled as simple DTOs, leaving the purpose interpretation and interaction responsibilities to the calling code (usually services). Usually, the business logic ends up being implemented in service classes, which can lead to code duplications, leaking of the business logic into other layers, missing or corrupted entity validations, and many more issues.

Ways to avoid an anemic domain model:

* Use private setters and expose methods to update data state.
* Always validate the state of entities - your entities must self-validate and not rely on API (contract) validation.
* Constructors without parameters are allowed to be only private as they are used by the ORM.
* Avoid primitive obsession.


### Primitive obsession

Primitive fields are basic built-in building blocks of a language such as integers, dates, strings, etc. Primitive Obsession is a programming style that heavily relies on primitives.
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

#### Single responsibility:

A class/method should have one and only one reason to change, meaning that a class/method should have only one job.

#### Open closed principle

Objects or entities should be open for extension but closed for modification. Class inheritance is not always the best way, coding to an interface is an integral part of SOLID.

### Liskov Substitution Principle

>  *Let q(x) be a property provable about objects of x of type T. Then q(y) should be provable for objects y of type S where S is a subtype of T.*

This means that every subclass or derived class should be substitutable for its base or parent class.

### Interface Segregation Principle

A client should never be forced to implement an interface that it doesn't use, or clients shouldn't be forced to depend on methods they do not use.
This means interfaces should be small, and should not contain methods not critically linked. If that is the case, consider splitting one interface into multiple smaller interfaces.

### Dependency inversion principle

Entities must depend on abstractions, not on concretions. The high-level modules must not depend on the low-level modules, but they both should depend on abstractions.
