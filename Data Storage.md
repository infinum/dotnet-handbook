We use several data storage types and different data access technologies in our projects. This part of the handbook will guide you through common practices in projects we are working on.

## Databases (relational)

This is a most common way of storing data. Relational databases provide a good way to store your entities and relationships between them. So far we have worked with SQL Server and PostgreSQL databases.  

To choose a database server (SQLServer or PostgreSQL) we need to keep some things in mind. Regular operations performance is quite similar and for really small applications the difference is minimal. SQL Server supports more functionalities (excellent data encryption, spatial types etc.) but it is also more expensive in higher tiers. Express version has a free license with memory limitations (1GB RAM, 10GB database size) but PostgreSQL license is completely free. From developer's perspective, it is a bit easier to work with SQL Server due to the Management studio which is a very good tool. 

So, when choosing, answer these questions:

* Is the client using a specific tech stack (or wants one)?
* What will be the expected database size in the lifetime of the application?
* What are the application requirements and required database functionalities?

### Entity Framework (EF Core)

Entity Framework is the most common mapper tool (ORM) used in .NET projects. It provides an excellent way to keep your models and database in sync through migrations. 

### DbContext

[Official documentation](https://docs.microsoft.com/en-us/ef/core/dbcontext-configuration/)

1. Create ApplicationDbContext
```c#
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
}
```

2. Register ApplicationDbContext in Startup.cs
```c#
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();

    services.AddDbContext<ApplicationDbContext>(
        options => options.UseSqlServer("Psst! It's a secret.json."));
}
```

### Configurations 

There are multiple ways to configure and build the model for your app. Although data annotations are also a good way to configure the models, we tend to use a fluent API approach. This provides more flexibility and support to the configuration.

Since we will be focusing mainly on examples in this handbook, you can read more in the [Official documentation](https://docs.microsoft.com/en-us/ef/core/modeling/).


```c#
// Fluent API example

public class User
{
    public string UserId { get; set; }
    public string Name { get; set; }
    public List<Comment> Comments { get; set; }
}

public class Comment
{
    public string CommentId { get; set; }    
    public string Title { get; set; }
    public string Content { get; set; }
    public string UserId { get; set; }
    public User User { get; set; }
}

public class UserEntityTypeConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.HasKey(k => k.UserId);        
        builder.Property(p => p.Name).IsRequired();
        
        builder
            .HasMany(p => p.Comments)
            .WithOne(p => p.User)
            .HasForeignKey(fk => fk.UserId)
    }
}

public class CommentEntityTypeConfiguration : IEntityTypeConfiguration<Comment>
{
    public void Configure(EntityTypeBuilder<Comment> builder)
    {
        builder.HasKey(k => k.CommentId);
        
        builder
            .Property(p => p.Title)
            .IsRequired();
        
        builder
            .Property(p => p.Content)
            .IsRequired();

        builder
            .HasOne(p => p.User)
            .WithMany(u => u.Comments)
            .HasForeignKey(fk => fk.UserId);
    }
}
```
&nbsp;

```c#
// Data annotations example

public class User
{
    [Key]
    public string UserId { get; set; }

    [Required]
    public string Name { get; set; }

    public List<Comment> Comments { get; set; }
}

public class Comment
{
    [Key]
    public string CommentId { get; set; }
    
    [Required]
    public string Title { get; set; }

    [Required]
    public string Content { get; set; }

    public string UserId { get; set; }

    public User User { get; set; }
}
```

&nbsp;

## Cloud Storages

Cloud provided new options when choosing storage for your applications. Although there are several provider options, we will focus mainly on Azure Storage in this handbook.


### Azure Storage

[Official documentation](https://docs.microsoft.com/en-us/azure/storage/common/storage-configure-connection-string)

For development we use tool called [Azure storage emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator). Fastest way to connect to your local Azure storage is by setting `UseDevelopmentStorage=true`.

This is equivalent of:
```
DefaultEndpointsProtocol=http;
AccountName=devstoreaccount1;
AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;
TableEndpoint=http://127.0.0.1:10001/devstoreaccount1;
```

And you can use it to connect to Blob, Queue and Table storages. If you want to see what data you added to your local storage, you can use [Azure Storage Expolorer](https://azure.microsoft.com/en-us/features/storage-explorer/#features).



#### Azure Blob Storage

Azure Blob storage is optimized for storing massive amounts of unstructured data like images, documents and other files. You can read more about it [here](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction).

Blob storage offers three types of resources:

##### Storage account:

- A unique namespace in Azure for your data
- If your storage account is named `myaccount`, then the default endpoint for blob is: `http://myaccount.blob.core.windows.net`

##### Containers:

- Similar to folders (we have files in a folder)
- Organizes a set of blobs (we have blobs in a container)
- You can have multiple containers (eg. photos, documents, etc.)

##### Blobs:

- Block blobs
- Append blobs
- Page blobs



```c#
// Configuration example

public interface IAzureBlobServiceClientFactory
{
    BlobServiceClient CreateBlobServiceClient();
}

public class AzureBlobOptions
{
    public string ConnectionString { get; set; }
}

public class AzureBlobServiceClientFactory : IAzureBlobServiceClientFactory
{
    private readonly IOptions<AzureBlobOptions> _options;
    
    public AzureBlobServiceClientFactory(IOptions<AzureBlobOptions> options)
    {
        _options = options;
    }
    
    public BlobServiceClient CreateBlobServiceClient()
    {
        return new BlobServiceClient(_options.Value.ConnectionString);
    }
}
```

```c#
// Usage example

public class TestService
{
    private readonly BlobServiceClient _client;

    public TestService(IAzureBlobServiceClientFactory azureBlobServiceClientFactory)
    {
        _client = azureBlobServiceClientFactory.CreateBlobServiceClient();
    }

    public async Task CreateContainerAsync(string containerName)
    {
        await _client.CreateContainerAsync(containerName);
    }
}
```

&nbsp;

#### Azure Table Storage

Azure Table Storage provides a way to store large amounts of structured data. This service is a NoSQL database. You can read more about it [here](https://docs.microsoft.com/en-us/azure/storage/tables/).

We must note that this is not a replacement for SQL database. For more information, please see [Understanding the differences between NoSQL and Relationl Databases.](https://docs.microsoft.com/en-us/azure/cosmos-db/relational-nosql)

Use it when you want to:
- Store data that doesn't require complex joins, foreign keys or any relationship.
- Store data which is denormalized.
- Have fast queries using a clustered index.

&nbsp;

## Cache

- Stores data as key value pairs.

Advantages
- Easy to use.
- Significantly improves the performance and scalability of an application by reducing the work required to generate content.

Disadvantages
- Potentially high memory usage (if not limited).
- Can have stale data (we must update cache).
- Possible concurrency issues (if not handled correctly).

### In-Memory Cache

[In-memory cache](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/response?view=aspnetcore-5.0) is the easiest way to add caching functionality to your application. It is used when you want to implement cache in single process. This means it won't be shared across multiple instances and will possibly cause issues in these kind of scenarios. It is also [Thread safe](https://docs.microsoft.com/en-us/dotnet/api/system.runtime.caching.memorycache?view=dotnet-plat-ext-5.0#thread-safety) which makes it an even better choice.

```c#
// Setup and usage example

public class HomeController : Controller
{
    private IMemoryCache _memoryCache;

    public HomeController(IMemoryCache memoryCache)
    {
        _memoryCache = memoryCache;
    }
}

public void ConfigureServices(IServiceCollection services)
{
    services.AddMemoryCache();
}
```