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
    [Required, MaxLength(200)]
    public string Title { get; set; } = default!;
    public bool IsActive { get; set; } = true;
    public DateTime CreatedUtc { get; set; } = DateTime.UtcNow;
}
```

### Create DBContext Model

```  cs
public class AppDBContext : DbContext
{
    public AppDBContext(DbContextOptions<AppDBContext> options) : base(options) { }

    public DbSet<Item> Items => Set<Item>();

    protected override void OnModelCreating(ModelBuilder b)
    {
        // -----------------------
        // Item
        // -----------------------
        b.Entity<Item>(e =>
        {
            e.ToTable("Item");
    
            e.Property(p => p.CreatedUtc)
                .HasDefaultValueSql("SYSUTCDATETIME()");
    
            e.Property(p => p.IsActive)
                .HasDefaultValue(true);

            e.HasIndex(p => p.Title)
                .IsUnique()
                .HasFilter("[Title] IS NOT NULL"); 
        });

}
```

### Package Manager Console

```  cs
Add-Migration "Initial Migration"
```

```  cs title="if no seeding"
Update-Database
```

### Seed Data
``` 
add-migration init 
```

``` cs
    public static class SeedData
    {
        public static void Initialize(AppDBContext db)
        {
            db.Database.Migrate();

            using var tx = db.Database.BeginTransaction();

            try
            {
                // -------------------------------
                // Items
                // -------------------------------
                if (!db.Items.Any())
                {
                    db.Units.AddRange(
                        new Item
                        {
                            Title = "Item1"
                        },
                        new Item
                        {
                            Title = "Item2"
                        }
                    );
                    db.SaveChanges();
                }

                var item1 = db.Items.First(u => u.Name.Contains("Item1"));
                var item2 = db.Items.First(u => u.Name.Contains("Item2"));

                tx.Commit();
            }
            catch
            {
                tx.Rollback();
                throw;
            }
        }
    }

```

``` cs title="Program.cs"
// ----------------------
// DB initial
// ----------------------
using (var scope = app.Services.CreateScope())
{
    var db = scope.ServiceProvider.GetRequiredService<AppDBContext>();
    SeedData.Initialize(db);
}
```

> After each changes: should run add-migration

## Database First 
### Package Manager Console

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


