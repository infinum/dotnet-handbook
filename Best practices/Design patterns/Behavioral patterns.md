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

