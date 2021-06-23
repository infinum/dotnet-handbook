# Fluent validation



### What is it?

We use fluent validation for determining "rules" which we want to include for our class, somewhat similar to Data annotations, but with more flexibility and control.



### How to use it:



**Firstly**, the Fluent Validation package should be referenced in your project, which you can simply do by using NuGet package manager or dotnet CLI command: 

`Install-Package FluentValidation`

or

`dotnet add package FluentValidation`



**Secondly**, find or write a class you would like to validate...

In our scenario, we would like to validate members of this DTO (data transfer objects) class so we can use this class as a "middleman" for saving data to the database. By doing that we don't expose our internal data structure to everybody 🙊.

```c#
public class LoginDetailsDto
{
    public string Username { get; set; }
    public DateTime DateTime { get; set; }
    public string AuthType { get; set; }
    public string IpAddress { get; set; }
}
```



**Thirdly**, create the validator...

Here we have an example Validator class where we validate the LoginDetailsDto class members. 

- The class must inherit from AbstractValidator <T> where T is the class you wish to validate
- Validation rules :
  - should be defined in the validator class constructor
  - property that we wish to validate should be passed a lambda expression
  - validators in the rule can be chained together and they can be followed by a message
    - there are a lot of **built-in validators** which you can check out on this [Built-in Validators](https://docs.fluentvalidation.net/en/latest/built-in-validators.html) page
    - you can also write your own, **custom validators**, by implementing the predicate validator which is accessed by using the **Must** method

- With function ValidateUsername we are making sure the username isn't already existing in the database because in our example we don't want duplicate usernames. The function must return Boolean data type.

```c#
using FluentValidaton;

public class LoginDetailsDtoValidator : AbstractValidator<LoginDetailsDto>
{
    public LoginDetailsDtoValidator()
    {
         RuleFor(loginDetailsDto => loginDetailsDto.Username)
             .NotEmpty()
             .WithMessage("Username is empty")
             .Must(ValidateUsername)
             .WithMessage("Username already exists");

         RuleFor(loginDetailsDto => loginDetailsDto.AuthType)
             .NotEmpty()
             .WithMessage("Authtype is empty");

         RuleFor(loginDetailsDto => loginDetailsDto.DateTime)
             .NotEmpty()
             .WithMessage("Date is empty");

         RuleFor(loginDetailsDto => loginDetailsDto.IpAddress)
             .NotEmpty()
             .NotEqual("1234");
    }
    
    public bool ValidateUsername(string username)
    {
         if(doesNameAlreadyExistInDatabase)
         {
             return false;
         }
         else
         {
             return true; 
         }
    }
}
```



**Later** we can use this validation where we see fit, in this case, we are validating if the input data is correct before we save it to the database. 

- To use the validator we should **register** it in Startup class in ConfigureServices method like this:
```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddFluentValidation(config =>
        {
            options.RunDefaultMvcValidationAfterFluentValidationExecutes = false;
            options.ValidatorOptions.PropertyNameResolver = FluentValidationResolvers.CamelCasePropertyNameResolver;
            options.RegisterValidatorsFromAssemblyContaining(typeof(LoginDetailsDtoValidator));
        });
}
```
  - for this to work the FluentValidation.AspNetCore package reference must be added
    `Install-Package FluentValidation.AspNetCore`
  - validation results are then added to ModelState, so we can use MVC's model binding infrastructure to validate the objects

```c#
public async Task CreateExternalLoginDetails(LoginDetailsDto googleUserInfo)
{
    if (ModelState.IsValid)
    {
        var loginDetails = new LoginDetails
        {
            Username = googleUserInfo.Username,
            AuthType = googleUserInfo.AuthType,
            IpAddress = googleUserInfo.IpAddress,
            DateTime = DateTime.UtcNow,
        };
        await _uow.LoginDetailsRepo.CreateAsync(loginDetails);
        await _uow.SaveChangesAsync();
    }
    else
    {
        foreach(var failure in results.Errors)
        {
            Console.WriteLine("Property" + failure.PropertyName + "failed validation.");
            Console.WriteLine("Error was: " + failure.ErrorMessage);
        }       
    }
}
```
- We check if the info is valid
- if it is valid, we wrap it into the correct type and save it to the database
- If it's not valid, fluent validation has its own property **Errors** which contains the collection of any validation failures, which we can then print out or log

- If we for some reason don't want to register the validator in services, we can also instantiate it manually
```c#
...
var validator = new LoginDetailsDtoValidator();
if (validator.Validate(googleUserInfo).IsValid)`{...}
...
```

#####Errors and exceptions
Combining all error messages into a single string:

- in the braces, you can pass a separator of your choice

```c#
validator.Validate(googleUserInfo).ToString("~");
```



You can also throw exceptions, by using ValidateAndThrow, like this:

```c#
using FluentValidation;
...
validator.ValidateAndThrow(googleUserInfo)`
```



Congratulations, we covered the basics, if you want to know more visit this link: [Fluent Validation Documentation](https://docs.fluentvalidation.net/)