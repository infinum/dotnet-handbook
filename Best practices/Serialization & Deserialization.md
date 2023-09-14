
## System.Text.Json

Serialization and deserialization are fundamental processes in software development, allowing for the conversion of objects to a format that can be easily stored or transmitted and then reconstructed later. In the .NET ecosystem, while there have been several libraries available for these operations, `System.Text.Json` has emerged as a modern,  high-performance option built directly into .NET Core.

For performance-critical scenarios  using `System.Text.Json` source generators  can provide significant improvements by generating optimized serialization code at compile-time.

### Basic Usage

To convert an object into its JSON representation:

```string jsonString = JsonSerializer.Serialize(yourObject);```

To convert a JSON string back into an object:

```YourObjectType obj = JsonSerializer.Deserialize<YourObjectType>(jsonString);```

### JsonSerializerOptions
`JsonSerializerOptions` is a class that provides a way to specify various serialization and deserialization behaviors. Here are some of the commonly used options:

**PropertyNameCaseInsensitive**: When set to `true`, property name matching during deserialization is case-insensitive. This can be useful when dealing with JSON from sources that don't maintain consistent casing.

```var options = new JsonSerializerOptions { PropertyNameCaseInsensitive = true };```

**WriteIndented**: Determines whether the output JSON should be pretty-printed with indentation, which can be useful for debugging or human readability.

```var options = new JsonSerializerOptions { WriteIndented = true };```

**DefaultIgnoreCondition**: Determines the condition under which a property will be ignored during serialization. For instance, you can set it to `JsonIgnoreCondition.WhenWritingNull` to ignore properties with `null` values.

```var options = new JsonSerializerOptions { DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull };```

**AllowTrailingCommas**: When set to `true`, the deserializer will tolerate trailing commas in the JSON.

```var options = new JsonSerializerOptions { AllowTrailingCommas = true };```

**MaxDepth**: Sets the maximum depth allowed when reading or writing JSON, protecting against deeply nested structures.

```var options = new JsonSerializerOptions { MaxDepth = 10 };```

**ReferenceHandler**: Determines how to handle references, which can be useful for dealing with circular references or preserving object references.

```var options = new JsonSerializerOptions { ReferenceHandler = ReferenceHandler.Preserve };```

**Converters**: You can add custom converters to handle specific serialization or deserialization scenarios that aren't covered by the default behavior. For instance, the `JsonStringEnumConverter` is commonly used if we want to represent enums as strings in the serialized output.

```options.Converters.Add(new MyCustomConverter());```

These are just some of the many options available in `JsonSerializerOptions`. By understanding and utilizing these options, developers can fine-tune the behavior of `System.Text.Json` to match the specific needs of their applications.

### Setting up JsonSerializerOptions globally

The `AddJsonOptions` extension method allows you to configure serialization settings globally for all controllers in an ASP.NET Core application.  You typically do this in the `Program.cs` while configuring MVC services. Example:

```  
services.AddControllers()  
 .AddJsonOptions(options => { ... });
 ```  

## Source Generators

The main drive behind Source Generators in System.Text.Json is performance. Traditional reflection-based serialization incurs runtime overhead, while Source Generators reduce this by generating code at compile-time. This leads to faster performance and reduced memory usage.

**Why we don't use htem as Default**

1.  **Flexibility and Customization**: Reflection offers dynamic behaviors hard to replicate with Source Generators.
2.  **Development Overhead**: Setting up Source Generators can add complexity.
3.  **Maturity Concerns**: Early adoption could have led to stability issues.

**When to Use Source Generators**

1.  **Performance-Critical Apps**: Where utmost speed is essential.
2.  **Stable Serialization Needs**: For well-defined, rarely changing serialization tasks.
3.  **Resource Limits**: In constrained environments, like IoT, where resources are precious.

This feature can operate in two primary modes: gathering metadata and optimizing serialization. By default it utilises both modes.

### Basic Setup:

To use source generation with default settings:
- Create a `partial` class that inherits from `JsonSerializerContext`.
- Mark the types you wish to serialize or deserialize with `JsonSerializableAttribute`.
- Invoke a `JsonSerializer` method that:
    - Accepts a `JsonTypeInfo<T>` instance, or
    - Accepts a `JsonSerializerContext` instance, or
    - Uses a a `JsonSerializerOptions` instance and you've set its `JsonSerializerOptions.TypeInfoResolver` property to the `Default` property of the context type


### Examples:

Consider a `WeatherForecast` class:
```  
public class WeatherForecast {  
    public DateTime Date { get; set; }
	public int TemperatureCelsius { get; set; }
    public string? Summary { get; set; }
}  
```  

To enable source generation for this class:
```  
[JsonSerializable(typeof(WeatherForecast))]  
public partial class WeatherForecastContext : JsonSerializerContext { }
 ```

Now you can utilise it like this:
```  
jsonString = JsonSerializer.Serialize(weatherForecast, typeof(WeatherForecast), WeatherForecastContext.Default);  
``` 
For ASP.NET Core Web API apps, you can use the `AddContext` method of `JsonSerializerOptions` to set up usage on controllers globally:
```  
services.AddControllers().AddJsonOptions(options =>  
	 options.JsonSerializerOptions.AddContext<MyJsonContext>());
 ```  


Sources:

- [Try the new System.Text.Json source generator](https://devblogs.microsoft.com/dotnet/try-the-new-system-text-json-source-generator)
- [How to use source generation in System.Text.Json](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation)
