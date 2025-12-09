# Migrate DB Context

## Migrate from MySQL to MsSQL

### Install required package
```csharp
Install Package Microsoft.EntityFrameworkCore.SqlServer
Install Package Microsoft.EntityFrameworkCore
Install Package Microsoft.EntityFrameworkCore.Tools
Install Package Microsoft.EntityFrameworkCore.Design
```

### Add SQL Server DB Context
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

### Add MySQL DB Context
```csharp
public class MySqlDbContext : DbContext
{
    private const string connectionString = @"Server=server.com;port=3306;Database=DB2;user=user;password=***";
    private readonly MariaDbServerVersion serverVersion = new(new Version(10, 5, 16));
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
        optionsBuilder.UseMySql(connectionString, serverVersion);
    }
}
```

### Add migration service
```csharp
public class MigrationToSqlServerService
{
    private readonly MySqlDbContext _mySqlDbContext;
    private readonly SqlServerDbContext _sqlServerDbContext;

    public MigrationToSqlServerService()
    {
        _mySqlDbContext = new MySqlDbContext();

        var sqlServerOptionsBuilder = new DbContextOptionsBuilder<SqlServerDbContext>();
        sqlServerOptionsBuilder.UseSqlServer("Server=.;Database=SimsaTest;User Id=sa;Password=QAZqaz@123;TrustServerCertificate=True");
        _sqlServerDbContext = new SqlServerDbContext(sqlServerOptionsBuilder.Options);
    }

    //public MigrationToSqlServerService(SimsaDb mySqlDbContext, SqlServerDbContext sqlServerDbContext)
    //{
    //    _mySqlDbContext = mySqlDbContext;
    //    _sqlServerDbContext = sqlServerDbContext;
    //}

    public async Task MigrateDataAsync()
    {
        await MigrateTableAsync<Arbeitgeber>(_mySqlDbContext.Arbeitgeber, _sqlServerDbContext.Arbeitgeber);
        await MigrateTableAsync<Arbeitgeberstammdaten>(_mySqlDbContext.Arbeitgeberstammdaten, _sqlServerDbContext.Arbeitgeberstammdaten);
        await MigrateTableAsync<Beratung>(_mySqlDbContext.Beratungen, _sqlServerDbContext.Beratungen);
        await MigrateTableAsync<GehaltsExtra>(_mySqlDbContext.GehaltsExtras, _sqlServerDbContext.GehaltsExtras);
        await MigrateTableAsync<Person>(_mySqlDbContext.Personen, _sqlServerDbContext.Personen);
        await MigrateTableAsync<Profil>(_mySqlDbContext.Profile, _sqlServerDbContext.Profile);
        await MigrateTableAsync<ProfilGehaltsExtra>(_mySqlDbContext.ProfilGehaltsExtras, _sqlServerDbContext.ProfilGehaltsExtras);
        await MigrateTableAsync<ProfilVorsorgevertrag>(_mySqlDbContext.ProfilVorsorgevertraege, _sqlServerDbContext.ProfilVorsorgevertraege);
        await MigrateTableAsync<Vorsorgevertrag>(_mySqlDbContext.Vorsorgevertraege, _sqlServerDbContext.Vorsorgevertraege);
    }

    private async Task MigrateTableAsync<TEntity>(DbSet<TEntity> source, DbSet<TEntity> destination) where TEntity : class
    {
        var tableName = _sqlServerDbContext.Model.FindEntityType(typeof(TEntity)).GetTableName();
        var primaryKeyProperty = _sqlServerDbContext.Model.FindEntityType(typeof(TEntity)).FindPrimaryKey().Properties.FirstOrDefault();

        var hasIdentityColumn = primaryKeyProperty != null && primaryKeyProperty.ValueGenerated == ValueGenerated.OnAdd;
        var isUniqueIdentifier = primaryKeyProperty != null && primaryKeyProperty.ClrType == typeof(Guid);

        await using var transaction = await _sqlServerDbContext.Database.BeginTransactionAsync();
        try
        {
            if (hasIdentityColumn && !isUniqueIdentifier)
            {
                await _sqlServerDbContext.Database.ExecuteSqlRawAsync($"SET IDENTITY_INSERT {tableName} ON");
            }

            var data = await source.AsNoTracking().ToListAsync();
            await destination.AddRangeAsync(data);
            await _sqlServerDbContext.SaveChangesAsync();

            if (hasIdentityColumn && !isUniqueIdentifier)
            {
                await _sqlServerDbContext.Database.ExecuteSqlRawAsync($"SET IDENTITY_INSERT {tableName} OFF");
            }

            await transaction.CommitAsync();
        }
        catch (Exception ex)
        {
            await transaction.RollbackAsync();
            throw;
        }
    }
```

- Add DBContext in program.cs (to run add-migration command)

```csharp
builder.Services.AddDbContext<SqlServerDbContext>(options =>
options.UseSqlServer("Server=.;Database=DB2;User Id=sa;Password=***;TrustServerCertificate=True"));
```

- Create SQL Server DB with the same name in ConnectionString

- Add migration with a specific DBContext When you have more than one context
- Run code in Package Manager Console
```
Add-Migration initialDB -Context SqlServerDbContext
```

- Update database with a specific DBContext
```
Update-Database -Context SqlServerDbContext
```

- Run this code to apply changes
```
var m = new MigrationToSqlServerService().MigrateDataAsync();
```
