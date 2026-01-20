# API Versioning

## Installation
1. Install required NuGet packages
```
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
dotnet add package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer
```

## Configuration
1. Configure API Versioning
    - Add this before 'builder.Services.AddControllers();'
    ``` cs
    builder.Services.AddApiVersioning(options =>
    {
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.AssumeDefaultVersionWhenUnspecified = true;

        // Report supported API versions in response headers
        options.ReportApiVersions = true;

        // Versioning strategy (URL-based)
        options.ApiVersionReader = new UrlSegmentApiVersionReader();
    });
    ```

    - Optional (recommended): API Explorer for Swagger
    ``` cs
    builder.Services.AddVersionedApiExplorer(options =>
    {
        options.GroupNameFormat = "'v'VVV";
        options.SubstituteApiVersionInUrl = true;
    });
    ```

## Usage  
- Update Controller
```
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/account")]
public sealed class AccountController : ControllerBase
{
    ...
}
```

- Multiple versioning
```
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/account")]
public sealed class AccountController : ControllerBase
{
    [HttpPost("login")]
    [MapToApiVersion("1.0")]
    public IActionResult LoginV1(...) { }

    [HttpPost("login")]
    [MapToApiVersion("2.0")]
    public IActionResult LoginV2(...) { }
}
```

- Alternative versioning strategies (choose ONE)
1. URL Segment (recommended)
```bash
/api/v1/account/login
```

``` cs title="Config"
options.ApiVersionReader = new UrlSegmentApiVersionReader();
```

2. Header-based
```bash
X-API-Version: 1.0
```

``` cs title="Config"
options.ApiVersionReader =
    new HeaderApiVersionReader("X-API-Version");
```

3. Query string
```bash
/api/account/login?api-version=1.0
```

``` cs title="Config"
options.ApiVersionReader =
    new QueryStringApiVersionReader("api-version");
```

> Middleware order (important)
```
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```
