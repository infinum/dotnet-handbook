Although it is not always clear when to use a certain design pattern, we should try to recognize the situation where a specific pattern can be used. For some of the application functionality we already have a predefined pattern. For the rest of the features it really depends on the situation and requirements.

Here are some of the design patterns we use:

## Creational patterns:

#### Factory method pattern

In this pattern we define an interface for object creation but the actual instantiation is done by subclass. This way we are creating an abstraction to a object creation. One of the main advantages is decoupling the code.

Lets say our application is receiving different events through a web socket and we need to handle each one in it's own way. Here is what would the implementation look like:

```c#
enum EventType
{
    Temperature,
    Fan
}

interface IEvent
{
    void Handle();
}

class TemperatureEvent : Event
{}

class FanEvent : Event
{}

class EventHandler
{
    public Event HandleEvent(EventType eventType, string eventMessage)
    {
        var event = GetEvent();
        event.Handle(eventMessage);
    }
    
    public IEvent GetEvent(EventType eventType)
    {
        switch(eventType)
        {
             case(EventType.Temperature):
                event = new TemperatureEvent();
                break;
             case(EventType.Fan):
                event = new FanEvent();
                break;
             ...
        }
    }
}
```



#### Builder pattern

This pattern allows you to build complex objects by using a step by step approach. This way we can easily control how is the object created.

```c#
class House
{
	public string Roof { get; set; }
    public string Walls { get; set; }
}

interface IHouseBuilder
{
    void BuildWalls();
    void BuildRoof();
    House GetHouse();
}

class HouseBuilder : IHouseBuilder
{
    var house = new House();
    
    public void BuildWalls()
    {
        house.Walls = "Walls";
    }
    
    public void BuildRoof()
    {
        house.Roof = "Roof";
    }
    
    public House GetHouse()
    {
        return house;
    }
}

// Usage
static void Main(string[] args)
{
    var housBuilder = new HouseBuilder();
    
    houseBuilder.BuildWalls();
    houseBuilder.BuildRoof();
    
    var house = houseBuilder.GetHouse();
}

```



## Structural patterns:

#### Repository and Unit of work (Facade)

One of the best definitions of repository is the one provided by Martin Fowler:

> *Conceptually, a Repository encapsulates the set of objects persisted in a data store and the operations performed over them, providing a more object-oriented view of the persistence layer. Repository also supports the objective of achieving a clean separation and one-way dependency between the domain and data mapping layers.*

This pattern can be used not matter what is used for accessing the data, Entity framework, table storage, external services etc. Since we are mostly using the Entity framework, we will provide an example using that data access approach.

Here is the simplified example of the pattern:

```c#
interface IRepository<T> where T : class
{
    Task<T> GetAsync(int id);
    
    // More methods would be added 
}

class Repository<T> : IRepository<T> where T : class
{
    private readonly DbSet<T> _dbSet;
    
    public async Task<T> GetAsync(int id)
    {
        return await _dbSet.FindAsync(id);
    }
}

interface IUnitOfWork
{
    IExampleRepository Examples { get; }
    Task SaveChangesAsync();
}

class UnitOfWork : IUnitOfWork
{
    private readonly ExampleDbContext _context;
    private IExampleRepository _exampleRepository;

    public UnitOfWork(ExampleDbContext context)
    {
        _context = context;
    }

    public IExampleRepository Examples 
        => _exampleRepository ??= new Repository<Example>(_context);

    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}

// Usage
class ExampleService
{
    private readonly IUnitOfWork _uow;
    
    public ExampleService(IUnitOfWork uow)
    {
     	_uow = uow;   
    }
    
    public async Task<Example> GetExample(int exampleId)
    {
        return await _uow.Examples.GetAsync(exampleId);
    }
}
```



## Behavioral Patterns:

#### Observer pattern

This pattern allows us to make a change of state in a single object and cascade this change to other observer objects. The object which initializes the change is inherited from the subject class and objects which are "subscribed" to that object are inherited from observer class. This works by subject class notifying other observers when a change of state happens in class which inherited the subject class. Observer objects can subscribe or unsubscribe from the subject at any time. 


The subject doesn't have to be aware of observer object's implementation, but it maintains a list of observers, which is then used to notify them, usually by calling one of their methods. 


Here we have a simple example of the observer pattern. Subject class allows observers to subscribe (or unsubscribe) to the objects that are inherited from subject class. By changing something in the login object, we automatically update the lists in NewsletterSubscription and DiscountSubscription class.  Every time a new Email is added, all objects which inherited the Observer class update themselves to the new Email.


```c#
abstract class Subject
{
    internal readonly List<IObserver> _observers;
    
    public void Subscribe(IObserver observer)
    {
        _observers.Add(observer);
    }
    
    public void Unsubscribe(IObserver observer)
    {
        _observers.Remove(observer);
    }
    
    public void NotifyObservers(string email)
    {
        foreach(var observer in _observers)
        {
            _observers.Update(email);
        }
    }
    
    public void AddEmail(string email)
    {
        NotifyObservers(email);
    }
}

public class Login : Subject 
{
    private string _email;
    
    public string GetEmail()
    {
        return _email:
    }
    
    public void SetEmail(string newEmail)
    {
        _email=newEmail;
        NotifyObservers(_email);
    }
}

public interface IObserver
{
    void Update(string email);
}

public class DiscountSubscription : IObserver
{
    public List<string> Emails { get; set; }
    
    public void Update(string email)
    {
        Emails.Add(email);
    }
}

public class NewsletterSubscription : IObserver
{
     public List<string> Emails { get; set; }

     public void Update(string email)
     {
         Emails.Add(email);
     }
}
```


