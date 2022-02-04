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
