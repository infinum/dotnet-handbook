We use fluent validation for determining "rules" which we want to include for our class, somewhat similar to Data annotations, but with more flexibility and control.

### How to use it:

**Firstly**, add the package...

Fluent Validation package should be referenced in your project, which you can simply do by using NuGet package manager or dotnet CLI command:  

```bash
Install-Package FluentValidation
```

or

```bash
dotnet add package FluentValidation
```

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
  - the property we want to validate should be passed a lambda expression
  - validators in the rule can be chained together and they can be followed by a message
    - there are a lot of **built-in validators** which you can check out on this [Built-in Validators](https://docs.fluentvalidation.net/en/latest/built-in-validators.html) page
    - you can also write your own, **custom validators**, by implementing the predicate validator which is accessed by using the **Must** method

- With the function `ValidateUsername` we are making sure the username isn't already existing in the database because in our example we don't want duplicate usernames. The function must return a Boolean data type.

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

- To use the validator we should **register** it in the Startup class in ConfigureServices method like this:

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

  - for this to work, FluentValidation.AspNetCore package reference must be added
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

- We check if the info is valid:
 - If it's valid, we wrap it into the correct type and save it to the database.
 - If it's not valid, Fluent Validation has its own property, **Errors**, which contains the collection of all validation failures, which we can then print out or log.

- If we for some reason don't want to register the validator in services, we can also instantiate it manually

```c#
...
var validator = new LoginDetailsDtoValidator();
if (validator.Validate(googleUserInfo).IsValid)`{...}
...
```

### Errors and exceptions
Combining all error messages into a single string:

- in the braces, you can pass a separator of your choice

```c#
validator.Validate(googleUserInfo).ToString("~");
```

You can also throw exceptions, by using ValidateAndThrow, like this:

```c#
using FluentValidation;
...
validator.ValidateAndThrow(googleUserInfo)
```  



### Configuration checker

 With fluent validation, we can validate **any** class, and there is nothing stopping us from using it to validate
configuration properties in the project. This is a very neat way of checking if all the configuration is set up before we fire up the solution.
The project will build normally, but when started it will throw an error and remind us of which configuration we are missing.
It's a really good solution for the dev-ops team when they are setting application configuration.
We will also bypass annoying unspecified errors which we usually get when the configuration is missing.

**How to do it?**


- Take all of the classes which are used in the IOptions pattern (this is an example)

```c#
public class SomeOption
{
    public string Option1 {get; set;}
}
```


- Create validators and specify the rules. Pay attention to include `.NotEmpty()` method, but you can add any validation you deem to be needed

```c#
public class SomeOptionValidator : AbstractValidator<SomeOption>
{
    public SomeOptionValidator()
    {
        RuleFor(c=>c.Option1).NotEmpty().WithMessage("{PropertyName} cannot be empty or null.");
    }
}
```

- Register the validators to validate at the start of the application

```c#
public static class ServiceCollectionExtensions
    {
        public static void BindOptions(
            this IServiceCollection services,
            IConfiguration config)
        {
            services.ConfigureOptionsSettings<SomeOption>(config);
        }

        public static void ConfigureOptionsSettings<T>(
            this IServiceCollection services,
            IConfiguration config,
            string? name = null) where T : class
        {
            var configurationSectionName = name ?? typeof(T).Name;

            var value = GetConfigurationValue<T>(config, configurationSectionName);

            var validationResult = IsValid(value, out string validationMessage);

            services.AddOptions<T>()
                .Bind(config.GetSection(configurationSectionName))
                .Validate(x => validationResult, validationMessage)
                .ValidateOnStart();
        }

        private static T GetConfigurationValue<T>(
            IConfiguration config,
            string configurationSectionName) where T : class
        {
            var value = config
                .GetSection(configurationSectionName)
                .Get<T>();

            if (value == default)
            {
                throw new Exception($"Validation error: Configuration section is missing for {configurationSectionName} settings.");
            }

            return value;
        }

        private static bool IsValid<T>(T value, out string message) where T : class
        {
            var validator = GenerateValidator<T>();
            var result = validator.Validate(value);
            message = FormatError(result);
            return result.IsValid;
        }

        private static BaseAbstractValidator<T> GenerateValidator<T>() where T : class
        {
            var type = FindValidatorImplementationType<T>();
            return Activator.CreateInstance(type) as BaseAbstractValidator<T>;
        }

        private static string FormatError(ValidationResult result)
            => $"Validation error: {string.Join(' ', result?.Errors?.Select(x => x.ErrorMessage))}";

        private static Type FindValidatorImplementationType<T>() where T : class
        {
            var type = Assembly
                .GetAssembly(typeof(BaseAbstractValidator<>))?
                .GetTypes()
                .FirstOrDefault(type => type.BaseType == typeof(BaseAbstractValidator<T>));
            if (type == default)
            {
                throw new ArgumentException($"Validator is not set for {typeof(T).Name} configuration.");
            }
            return type;
        }
    }
```

- Call the first static method in the Startup class

```c#
public class Startup
{
    ...
    public void ConfigureServices(IServiceCollection services)
    {
        ...
        services.BindOptions(Configuration);
        ...
    }
}
```

- Comment out all of the used properties from the appsettings.json, as the properties will be read from
it for configuration and the checker will not work
- Done

**Disclaimer:** this will validate the property as well as the configuration section!

Congratulations, we covered the basics, if you want to know more visit this link: [Fluent Validation Documentation](https://docs.fluentvalidation.net/)
