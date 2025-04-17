# Application Settings


## Powershell env. Config
``` bash title="Set Environment Variable"
[System.Environment]::SetEnvironmentVariable("DB_PASSWORD", "MySecret123", "Machine")
```

> [!CAUTION]
> Run the PowerShell as Administrator
> Restart the IDE after setting the **Environment** values

## Set appsettings.json
``` cs
"ConnectionStrings": {
  "DefaultConnection": "Server=...;User Id=...;Password=%DB_PASSWORD%;"
},
"ExAuthentication": {
    "Facebook": {
      "AppId": "facebook",
      "AppSecret": "facebook"
    },
    "Google": {
      "ClientId": "",
      "ClientSecret": ""
    }
  }
```

## Set Program.cs 
``` cs title="program.cs"
// Register the IConfiguration instance which 'secrets.json', 'appsettings.json', 'EnvironmentVariables' binds against.
//----------------------------------------
var configuration = new ConfigurationBuilder()
        .SetBasePath(Environment.CurrentDirectory)
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddUserSecrets<Program>(optional: true, reloadOnChange: true)
        .AddEnvironmentVariables()
            .Build();

builder.Services.AddSingleton<IConfiguration>(configuration);

//Configure DB Context
//----------------------------------------
builder.Services.AddDbContext<DBContext>(options =>
{
    options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")
            .Replace("%DB_PASSWORD%", Environment.GetEnvironmentVariable("DB_PASSWORD")));
}, ServiceLifetime.Scoped);
```

## Read appsettings.json 
``` cs title="usage"
configuration.GetSection("ExAuthentication:Google:ClientId").Value
```

## CMD env. Config
```
setx ConnectionStrings__DBConnection "Server=.;Database=DB;User Id=sa;Password=***;TrustServerCertificate=true;"

setx ExAuthentication__Facebook__AppId "facebook"
setx ExAuthentication__Facebook__AppSecret "facebook"

setx ExAuthentication__Google__ClientId "ClientId"
setx ExAuthentication__Google__ClientSecret "ClientSecret"
```

## Powershell env. Config
```
# If you need to set these variables for the current user only, replace 'Machine' to 'User'
# -----------------------------------
[System.Environment]::SetEnvironmentVariable('ConnectionStrings__DBConnection', 'Server=.;Database=DB;User Id=sa;Password=***;TrustServerCertificate=true;', [System.EnvironmentVariableTarget]::Machine)

[System.Environment]::SetEnvironmentVariable('ExAuthentication__Facebook__AppId', 'facebook', [System.EnvironmentVariableTarget]::Machine)
[System.Environment]::SetEnvironmentVariable('ExAuthentication__Facebook__AppSecret', 'facebook', [System.EnvironmentVariableTarget]::Machine)

[System.Environment]::SetEnvironmentVariable('ExAuthentication__Google__ClientId', 'ClientID', [System.EnvironmentVariableTarget]::Machine)
[System.Environment]::SetEnvironmentVariable('ExAuthentication__Google__ClientSecret', 'ClientSecret', [System.EnvironmentVariableTarget]::Machine)
```

## User Secrets file
```
install-package Microsoft.Extensions.Configuration.UserSecrets

  <PropertyGroup>
    <UserSecretsId>SecretId</UserSecretsId>
  </PropertyGroup>

add settings in Manage user secrets on contextmenu of each project
```
