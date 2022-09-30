#### Decorator

We use Decorator Pattern when we need to attach additional responsibilities to an individual objects dynamically. The decorator conforms to the interface of the component it decorates so that its presence is transparent to the component's clients. The decorator forwards requests to the component and may perform additional actions. Transparency lets us nest decorators recursively.

It is important to notice that we are subclassing the abstraction of component we are decorating in order to have the correct type, not to inherit its behavior. The behavior comes in through the composition of decorators with the base components as well as other decorators.

```c#
interface Car
{
    string Model { get; }
    int Price { get; }
}


class Porche : Car
{
    public string Model
    {
        get { return "Porche"; }
    }

    public double Price
    {
        get { return 399999; }
    }
}


abstract class CarDecorator : Car
{
    private Vehicle _vehicle;

    public CarDecorator(Vehicle vehicle)
    {
        _vehicle = vehicle;
    }

    public string Model
    {
        get { return _vehicle.Model; }
    }

    public double Price
    {
        get { return _vehicle.Price; }
    }

}

class Discounted : CarDecorator
{
    public Discounted(Car car) : base(car) { }

    public int Percentage { get; set; }

    public double Price
    {
        get { return _vehicle.Price * (1 - Percentage); }
    }
}
```

#### Adapter

The Adapter pattern is a structural design pattern that we use when we need objects with incompatible interfaces to collaborate. Adapters convert the interface of a class into another interface clients expect. 

```c#
class Adapter : PolarVector
{
    private readonly CartesianVector _v;

    Adapter(CartesianVector v)
    {
	  _v = v;
    }

    public double Magnitude = get { Sqrt(Pow(_v.X, 2) + Pow(_v.Y, 2)); }
    public double Angle = get { 1 / Tan(_v.Y / _v.X); }

}

// adaptee
class CartesianVector
{
    ...
    public double X { get; };
    public double Y { get; };
}

// target
class PolarVector
{
    ...
    public double Angle { get; };
    public double Magnitude { get; };
}
```



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



#### Strategy pattern

By using this pattern we enable a selection of needed algorithms at runtime. For instance, validation, which is dependent on the incoming type of data. We don't know beforehand which algorithm will be needed, but we can develop a few strategies and use them when needed. Algorithms should be defined in a way that they could be used interchangeably.

This pattern allows us to decouple the code, encapsulate each algorithm, and lets the algorithm be independent of the clients which use it.



```c#
public interface IStrategy
{
    string Payment();
}

public class CardPayment : IStrategy
{
    public string Payment()
    {
        return "Paid by a card";
    }
}

public class CashPayment : IStrategy
{
    public string Payment()
    {
        return "Paid with cash";
    }
}

public class PaymentMethod
{
    public IStrategy Strategy { get; set; }

    public void GetPaymentMethod()
    {
        Console.WriteLine(Strategy.Payment());
    }
}
```
