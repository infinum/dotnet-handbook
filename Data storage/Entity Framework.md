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
        builder.Property(p => p.Title).IsRequired();
        builder.Property(p => p.Content).IsRequired();

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

### Migrations

In the example above, we created User and Comment classes with **Entity Type Configuration** which describes how the data should "behave". Now, if we want to take a snapshot of that data with the configuration and translate it to the database automatically, Entity framework has the easy solution for us! **Migrations** help us create entity types by reverse engineering the schema of a database. 

First step is to go into your **Startup** Class and configure the Database Context in a way it knows where to look for configurations for the migrations, we will just upgrade the code we had above (we don't need to do this if we are using Data Annotations, but we avoid using them):

```c#
services.AddDbContext<YourTypeOfDbContext>(options =>
       options.UseSqlServer("Psst! It's a secret.json"),
       x => x.MigrationsAssembly("NameOfTheClassLibraryWhereConfigIsLocated")));
```


Now, lets try to create our first migration, we will migrate our User and Comment classes with their configuration to the the database.

There are two ways to do that (look for them in the search tab if they are not in your main window): 

	**.NET Core CLI** 
	
			`dotnet ef migrations add InitialCreate`
	
	**Package Manager Console**
	
			`Add-Migration InitialCreate`


After we run one of these commands EF Core will do all the heavy lifting for us. It will create a folder named Migrations in your project and generate the migration class there. It should look something like this:

```c#
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "User",
            columns: table => new
            {
                UserId = table.Column<string>(type: "nvarchar(max)", nullable: false),
                Name = table.Column<string>(type: "nvarchar(max)", nullable: false),
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_User", x => x.UserId);
            });
        
        migrationBuilder.CreateTable(
            name: "Comment",
            columns: table => new
            {
               CommentId = table.Column<string>(type: "nvarchar(max)", nullable: false),
               Title = table.Column<string>(type: "nvarchar(max)", nullable: false),
               Content = table.Column<string>(type: "nvarchar(max)", nullable: false),
               UserId = table.Column<string>(type: "nvarchar(max)", nullable: false),
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Comment", x => x.CommentId);
                table.ForeignKey("User", x => x.UserId, cascadeDelete: true)
                table.Index(x => x.UserId)
            });
    }
}
```

Nice, now that you have the Migration set up, there is only one thing to do, update it, so it can create your database schema from the migration. Again we have two options:

	**.NET Core CLI** 
	
			`dotnet ef database update`
	
	**Package Manager Console**
	
			`Update-Database`


And that's it, the database tables are created, you can check them in the Server Explorer! It's very easy and you don't need to use a single line of SQL!


##### What to do when we updated the class and want to apply the changes to the database? 

Rinse and repeat, create the migration again as we did above and update it, the whole process is handled by the Entity framework core for you! The new model is now compared to the snapshot of the old model, EF detects which column was added or changed and scaffolds an appropriate migration for you.
