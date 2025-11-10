# Entity Framework

## Install Packages 
```  cs
Microsoft.EntityFrameworkCore
```
```  cs
Microsoft.EntityFrameworkCore.Tools
```
``` cs
Microsoft.EntityFrameworkCore.SqlServer
```
## Add DI
```  cs
builder.Services.AddDbContext<AppDBContext>(options =>
  options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnectionString"))
);
```

## Code First 
### Create Model
```  cs
public class Item
{
    public int ID { get; set; } 
    public string Title { get; set; }
}
```  
### Create DBContext Model
```  cs
public class AppDBContext : DbContext
{
    public AppDBContext(DbContextOptions<AppDBContext> options) : base(options) { }

    public DbSet<Item> Items { get; set; }
}
```

### Package Manager Console CMD
```  cs
Add-Migration "Initial Migration"
```
```  cs
Update-Database
```

## Database First 
### Package Manager Console CMD
```  cs
Scaffold-DbContext "Connection String" Microsoft.EntityFrameworkCore.SqlServer -ContextDir DataFolder -OutputDir Models -DataAnnotation
```

## Usage 
```  cs
private readonly AppDBContext _context;
```

## Linq to SQL 
``` cs
using (var dbContext = new DbContext())
{
  var query = dbContext.Users
     .Where (u => u.Age>5)
     .Orderby (u => u.Name);
  string query = query.ToQueryString();
  Console.WriteLine(query);
}
```

## Global Query Filter
- **OnModelCreating** is a method for defining the model structure:
    - Table mappings
    - Relationships
    - Constraints
    - Default values
    - **Query filters** ✅

- **OnConfiguring** is a method for configuring the database connection.
    
```  cs title="BlogContext.cs"
public class BlogContext : DbContext
{
  protected override void OnModelCreating(ModelBuilder modelBuilder)
  {
    modelBuilder.Entity<Blog>()
        .HasQueryFilters(b => !b.IsArchived);
  }
}
```
``` cs title="To disable it"
val allBlogs = await _context.Blogs.IgnoreQueryFilters().ToListAsync();
```
