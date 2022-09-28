## What are Source Generators?

According to the official [documentation](https://docs.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview), a source generator is a part of the .NET Compiler Platform ("Roslyn") SDK which allows developers to generate source files during compilation and add those files to the compilation object.

For this to be possible, source generators can retrieve a compilation object which contains all the code a developer has written. The most common use case for source generators is code analysis during compilation, after which more code is generated and added to the compilation object. It can also shift the controller discovery phase from startup to compile time, reducing startup time. Another usage of source generators can be in ASP.NET Core applications routing where it can be strongly typed with the necessary strings being generated as a compile-time detail. This approach could prevent a mistyped string literal leading to a request not hitting the correct controller.

The image below shows the process which happens during compilation if a developer uses source generators.

| ![sourceGenerator](/resources/source-generator-visualization.png) |
|:--:|
| Figure 1 - Visualization of compilation that includes a Source Generator |

There are multiple advantages of using source generators which include better performance because the source generator is analyzing code during the compilation. An alternative to this approach is runtime reflection which is used during the startup of an application and therefore can prolong the startup time. Another advantage is the possibility to catch errors during compile time instead of catching them during runtime.

## Project setup

Projects that contain source generators are just ordinary class libraries that require some adjustments. Here are some basic steps that can be used for source generator project setup.

1. Create a new class library project with .NET Standard 2.0 target framework
2. Add NuGet packages *Microsoft.CodeAnalysis.Analyzers* and *Microsoft.CodeAnalysis.CSharp*

Here is an example of what a project file should look like after the first two steps:

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netstandard2.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.0.1" PrivateAssets="all" />
    <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.3">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>

</Project>
```

3. Create a *ExampleSourceGenerator.cs* file which implements *Microsoft.CodeAnalysis.ISourceGenerator* interface and annotate it with *Microsoft.CodeAnalysis.GeneratorAttribute*

```c#
using Microsoft.CodeAnalysis;

namespace SourceGenerator
{
    [Generator]
    public class ExampleSourceGenerator : ISourceGenerator
    {
        public void Execute(GeneratorExecutionContext context)
        {
            // Code generation goes here
        }

        public void Initialize(GeneratorInitializationContext context)
        {
            // If required, initialization goes here
        }
    }
}
```

After these three steps your source generator is configured and to do something with it, you need to implement *Execute* method. Implementation of *Initialize* method is optional because it is not always required to initialize a source generator.


## Debugging

If you've used source generators then you must know that it can be quite challenging to test the generator's output and find out what kind of error it throws.

Fortunately, Visual Studio comes to the rescue with its possibility to debug source generators after some easy steps:

1. Install .NET Compiler Platform SDK 
   
    In Visual Studio Installer choose to Modify the Visual Studio version which has a source generator project, and install Visual Studio extension development under tab Workloads > Other Toolsets.

    | ![visualStudioInstallerHome](/resources/visual-studio-installer-home.png) |
    |:--:|
    | Figure 2 - Visual Studio Installer |

    | ![visualStudioModify](/resources/visual-studio-modify.png) |
    |:--:|
    | Figure 3 - Visual Studio modifying window |

2. Add IsRoslynComponent property in source generator project file:
   
    ```xml
    <Project Sdk="Microsoft.NET.Sdk">
        <PropertyGroup>
            <TargetFramework>netstandard2.0</TargetFramework>
            <IsRoslynComponent>true</IsRoslynComponent>
            <Nullable>enable</Nullable>
            <LangVersion>latest</LangVersion>
        </PropertyGroup>
        <ItemGroup>
            <PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.0.1" PrivateAssets="all" />
            <PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.3.3">
                <PrivateAssets>all</PrivateAssets>
                <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
            </PackageReference>
        </ItemGroup>
    </Project>
    ```

3. Add a reference to source generator in a project that will be using it (if it already isn't added):
   
    ```xml
    <ItemGroup>
        <ProjectReference Include="..\JsonApi.SourceGenerator\JsonApi.SourceGenerator.csproj" OutputItemType="Analyzer" ReferenceOutputAssembly="true" />
    </ItemGroup>
    ```

4. Add debug configuration using the following steps:
   
   1. Right-click on the project that contains the source generator
   2. Click `Properties`
   3. Click `Debug`
   4. Click `Open debug launch profiles UI`
      | ![debugProperties](/resources/debug-properties.png) |
      |:--:|
      | Figure 4 - Debug properties window |
   5. Delete existing profiles
      | ![deleteProfile](/resources/delete-existing-profile.png) |
      |:--:|
      | Figure 5 - Delete existing profile |
   6. Add new `Roslyn Component` profile
      | ![createProfile](/resources/create-roslyn-component-profile.png) |
      |:--:|
      | Figure 6 - Add new profile |
   7. Choose the target project from a list of projects
      | ![chooseTargetProject](/resources/choose-target-project.png) |
      |:--:|
      | Figure 7 - Choose the target project |
   8. **Optional**: Rename the debug profile so you can easily find and use it later
   9. Restart Visual Studio and set your source generator project as a startup project

| ![endResult](/resources/end-result.png) |
|:--:|
| Figure 7 - Debug profile after creation |

In the end add some breakpoints to the source generator code, run the project in debug mode, and resolve the issues that were bothering you in the first place or just explore how your source generator works in the background.

Sources:

- [Introducing C# Source Generators](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/)
- [Introducing C# Source Generators](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/)
- [dotnet-how-to-debug-source-generator-vs2022](https://github.com/JoanComasFdz/dotnet-how-to-debug-source-generator-vs2022)
- [Source Generators](https://docs.microsoft.com/en-us/dotnet/csharp/roslyn-sdk/source-generators-overview)
- [Debugging C# Source Generators with Visual Studio 2019 16.10](https://stevetalkscode.co.uk/debug-source-generators-with-vs2019-1610)