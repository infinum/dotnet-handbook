In .NET Core 3.0 and later, certain APIs (e.g., those for ASP.NET Core) are part of the **shared framework** and are no longer distributed as standalone NuGet packages. To use these APIs in your class library, you need to configure your library to target the appropriate framework. Here's how:

---

### **Step 1: Edit the .csproj File**

You need to specify the **FrameworkReference** for `Microsoft.AspNetCore.App` in your class library's `.csproj` file.

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <FrameworkReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>
</Project>
```

---

#### What Does Adding FrameworkReference Do?

specific APIs included in the ASP.NET Core shared framework. These APIs are provided as part of the runtime, rather than being distributed via NuGet packages.

That said, in most cases, you don’t need to add a **FrameworkReference** manually. When a class library is used in an ASP.NET Core application (e.g., API project), the application already references `Microsoft.AspNetCore.App` and makes these APIs available to the library.

---

#### So, When Do You Need FrameworkReference?

You only need to explicitly add a **FrameworkReference** in the class library if your library is intended to be used independently (e.g., for unit tests or another non-ASP.NET Core project) and directly depends on ASP.NET Core APIs, you must add a **FrameworkReference**. Without it, the library won’t compile because it won’t have access to ASP.NET Core types.

---

### **Step 2: Build and Use the Class Library**

Once you’ve added the **FrameworkReference**, write your library code and include any necessary ASP.NET Core namespaces. For example:

```c#
using Microsoft.AspNetCore.Http; // For working with HTTP context
using Microsoft.Extensions.DependencyInjection; // For dependency injection
// more usings...

public class ExampleService
{
    // service logic
}
```

When the library is ready, reference it in your ASP.NET Core application like any other library.

---

This small addition to the .csproj ensures your library is properly configured to use ASP.NET Core APIs in a self-contained and portable way, regardless of where it’s used.