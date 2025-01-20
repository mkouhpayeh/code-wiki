## Migrate from MySQL to MsSQL

## Install required package
```csharp
Install Package Microsoft.EntityFrameworkCore.SqlServer
```

## Add SQL Server DB Context
```csharp
public class SqlServerDbContext : DbContext
{
    private const string connectionString = @"Server=.;Database=DB1;User Id=sa;Password=***;TrustServerCertificate=True";

    public SqlServerDbContext(DbContextOptions<SqlServerDbContext> options) : base(options) { }

    public DbSet<Arbeitgeber> Arbeitgeber { get; set; }
    public DbSet<Arbeitgeberstammdaten> Arbeitgeberstammdaten { get; set; }
    public DbSet<Beratung> Beratungen { get; set; }
    public DbSet<GehaltsExtra> GehaltsExtras { get; set; }
    public DbSet<Person> Personen { get; set; }
    public DbSet<Profil> Profile { get; set; }
    public DbSet<ProfilGehaltsExtra> ProfilGehaltsExtras { get; set; }
    public DbSet<ProfilVorsorgevertrag> ProfilVorsorgevertraege { get; set; }
    public DbSet<Vorsorgevertrag> Vorsorgevertraege { get; set; }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        if (!optionsBuilder.IsConfigured)
        {
            optionsBuilder.UseSqlServer(connectionString);
        }
    }
}
```

