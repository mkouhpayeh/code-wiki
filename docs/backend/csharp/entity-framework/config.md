# Configuration 

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
        .HasQueryFilter(b => !b.IsArchived);
  }
}
```

``` cs title="To disable it"
val allBlogs = await _context.Blogs.IgnoreQueryFilters().ToListAsync();
```


## Save Behaviors
EF Core has two different “phases” of saving:
    - BeforeSaveBehavior → applies on INSERT
    - AfterSaveBehavior → applies on UPDATE

1. 🟦 BeforeSaveBehavior
(Controls how EF handles the property BEFORE the entity is first saved — INSERT)
Options:

    - ✔️ Save ➡️ EF writes the value during INSERT

    - ✔️ Ignore ➡️ EF does not write property value on INSERT

    - ✔️ Throw ➡️ EF throws exception if the property has a value on INSERT ❌ throws InvalidOperationException

2. 🟩 AfterSaveBehavior
Controls how EF handles property AFTER the entity is saved — UPDATE)
Options:

    - ✔️ Save ➡️ EF updates the value 

    - ✔️ Ignore ➡️ EF ignores changes

    - ✔️ Throw ➡️ EF throws exception on modification ❌ throws InvalidOperationException

- To prevent EF from updating the property after the entity is saved

> entity.Code = "ABC123";
context.SaveChanges(); // value written to DB

> entity.Code = "XYZ999";
context.SaveChanges(); // ignored — DB value is not updated

```
builder.Property(p=>p.Code)
    .IsRequired()
    .HasMaxLength(10);

builder.Property(p=>p.Code)
    .Metadata.SetAfterSaveBehavior(PropertySaveBehavior.Ignore); // when the property should be set once and never changed again.
```
