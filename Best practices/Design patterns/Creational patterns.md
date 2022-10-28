#### Factory method pattern

In this pattern, we define an interface for object creation but the actual instantiation is done by a subclass. This way we are creating an abstraction to object creation. One of the main advantages is decoupling the code.

```c#
class Taxes
{
    public const int GREENTAX = 1.15;
}

abstract class Vehicle
{
    int price;
}

class Car: Vehicle
{
}

class Bike : Vehicle
{
}

abstract class VehicleCatalog
{
    public List<IVehicle> Items;

    public void AddToCatalog(int price);
    {
        var newVehicle = CreateVehicle(price);
        Items.Add(newVehicle);
    }

    // factory method
    public abstract IVehicle CreateVehicle(int price);
}

class BikeCatalog : VehicleCatalog
{
    public override IVehicle CreateVehicle(int price)
    {
        return new Bike()
	  {
		this.price = price;
	  }
    }
}

class CarCatalog : VehicleCatalog
{
    public override IVehicle CreateVehicle(int price)
    {
        return new Car()
	  {
		this.price = price * GREENTAX;
	  }
    }
}
```



#### Abstract Factory

We use the Abstract Factory Pattern when we need an interface for creating families of related or dependent objects without specifying their concrete classes. This family of related objects is often designed to be used together so with Abstract Factory Pattern we enforce this constraint. Most often the methods of an Abstract Factory are implemented as factory methods.

To use the factory, you instantiate one and pass it into some code that is written against the abstract type. So, like in the Factory Method pattern, clients are decoupled from the actual concrete classes they use.

```c#
interface ICarPartsFactory
{
    IEngine CreateEngine();
    ITire CreateTire();
}

class PorcheFactory : ICarPartsFactory
{
    public IEngine CreateEngine();
    {
        return new BoxerEngine();
    }

    public ITire CreateTire()
    {
        return new PerformanceTires();
    }
}

class JeepFactory : ICarPartsFactory
{
    public IEngine CreateEngine()
    {
        return new V6Engine();
    }

    public ITire CreateTire()
    {
        return new AllTerrainTires();
    }
}

interface IEngine { ... }
class BoxerEngine : IEngine { ... }
class V6Engine : IEngine { ... }

interface ITire { ... }
class PerformanceTires : ITire { ... }
class AllTerrainTires : ITire { ... }


//client
class CarAssembler
{
    private IEngine _productA;
    private ITire _productB;

    public Assembly(ICarPartsFactory factory)
    {
        _productA = factory.CreateEngine();
        _productB = factory.CreateTire();
	  ...
    }
}
```



#### Builder pattern

This pattern allows you to build complex objects by using a step-by-step approach. This way we can easily control how is the object created.

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
    var houseBuilder = new HouseBuilder();

    houseBuilder.BuildWalls();
    houseBuilder.BuildRoof();

    var house = houseBuilder.GetHouse();
}

```
