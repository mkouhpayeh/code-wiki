# Connection String

## SQL Server
> [!NOTE]
> Use a comma to specify a port number with SQL Server: `mycomputer.test.xxx.com,1234`  
It's not necessary to specify an instance name when specifying the port.    

``` cs title="AppSettings"
"ConnectionStrings": {
      "DBConnection": "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;Encrypt=True;TrustServerCertificate=False;"
}
```

> [!NOTE]
>  Make sure the SQL Server uses a valid SSL certificate:
*  Get a Valid SSL Certificate
*  Assign the Certificate to SQL Server:  SQL Server Configuration Manager > SQL Server Network Configuration > Protocols for MSSQLSERVER (or instance name) > Right-click "Properties" > Certificate tab
*  Enable Force Encryption (Optional but Recommended):  SQL Server Configuration Manager > under Protocols for MSSQLSERVER > Right-click > Properties > Under the Flags tab, set Force Encryption to Yes

``` cs title="Program.cs"
//Configure DB Context
//----------------------------------------
builder.Services.AddDbContext<DBContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("DBConnection"));
});
```

## PostgreSQL

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
