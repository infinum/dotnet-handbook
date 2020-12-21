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

