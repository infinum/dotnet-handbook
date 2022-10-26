#### Strategy pattern

By using this pattern we enable a selection of needed algorithms at runtime. For instance, validation is dependent on the incoming type of data. We don't know beforehand which algorithm will be needed, but we can develop a few strategies and use them when needed. Algorithms should be defined in a way that they could be used interchangeably.

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




#### Observer pattern

This pattern allows us to make a change of state in a single object and cascade this change to other observer objects. The object which initializes the change is inherited from the subject class and objects which are "subscribed" to that object are inherited from the observer class. This works by having the subject class notify other observers when a change of state happens in a class that inherited the subject class. Observer objects can subscribe or unsubscribe from the subject at any time.


The subject doesn't have to be aware of the observer object's implementation, but it maintains a list of observers, which is then used to notify them, usually by calling one of their methods.


Here we have a simple example of the observer pattern. The subject class allows observers to subscribe (or unsubscribe) to the objects that are inherited from the subject class. By changing something in the login object, we automatically update the lists in NewsletterSubscription and DiscountSubscription classes.  Every time a new Email is added, all objects which inherited the Observer class update themselves to the new Email.

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
        _email = newEmail;
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




#### Chain of Responsability

With the Chain of Responsibility pattern, we avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request.

To implement this pattern we create a chain of objects to examine requests. Each object in turn examines a request and either handles it or passes it on to the next object in the chain. A client that sends request doesn’t have to know the chain’s structure nor keep direct references to its members.

```c#
abstract class CarService
{
    protected CarService _successor;

    public abstract void Service(Car car);

    public void SetSuccessor(CarService successor)
    {
        _successor = successor;
    }
}

class EngineService : CarService
{
    public override void Service(Car car)
    {
        if (EngineDiagnostics(car))
        	EngineRepair(car);
        else if (_successor != null)
            _successor.Service(Car car);
    }

    private bool EngineDiagnostics(Car car){...}
    private void EngineRepair(Car car){...}
}

class TransmissionService: CarService
{
    public override void Service(Car car)
    {
        if (TransmissionDiagnostics(car))
        	TransmissionRepair(car);
        else if (_successor != null)
            _successor.Service(request);
    }

    private bool TransmissionDiagnostics(Car car){...}
    private void TransmissionRepair(Car car){...}
}
```



#### Template Method

Template method pattern is all about encapsulating an algorithm. It defines the skeleton of an algorithm in the superclass but lets subclasses override specific steps of the algorithm without changing its structure.

The template method defines a sequence of steps, each represented by a method. Concrete methods already have some default implementation, but still can be overridden if needed, while abstract methods must be implemented by every subclass. There is also a hook, which is a concrete method with an empty body. A template method would work even if a hook isn’t overridden. Usually, hooks are placed before and after crucial steps of algorithms, providing subclasses with additional extension points for an algorithm.

```c#
abstract class Worker
{
    // template method
    public final void MorningRoutine()
    {
        WakeUp();
        EatBreakfast();
        GoToWork();
    }

    public void WakeUp() {...};
    public void EatBreakfast() {...};
    public abstract void GoToWork();

}

class FireFighter : Worker
{
    public override void GoToWork() {...}
}
```
