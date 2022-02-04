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
