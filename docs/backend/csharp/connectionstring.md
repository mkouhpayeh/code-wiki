# Connection String

## SQL Server
!!! note ""    

    Use a comma to specify a port number with SQL Server: `mycomputer.test.xxx.com,1234`  
    It's not necessary to specify an instance name when specifying the port.

## PostgreSQL
!!! note ""    

``` cs title="AppSettings"
"ConnectionStrings": {
  "DBConnection": "User ID=postgres;Password=***;Host=localhost;Port=5432;Database=DB1;Pooling=true;"
}
```
``` cs title="Program.cs"
//Configure DB Context
//----------------------------------------
builder.Services.AddDbContext<DBContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("DBConnection"));
}, ServiceLifetime.Scoped);
```
