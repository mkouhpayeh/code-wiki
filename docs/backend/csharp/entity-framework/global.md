# Globalization 

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


## Initialization
```
builder.Property(p=>p.Code)
    .IsRequired()
    .HasMaxLength(10);

nuilder.Property(p=>p.Code)
    .Metadata.SetAfterSaveBehavior(PropertySaveBehavior.Ignore);
```


``` cs title="To disable it"
val allBlogs = await _context.Blogs.IgnoreQueryFilters().ToListAsync();
```
