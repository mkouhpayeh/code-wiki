# API Versioning

---

## Installation
1. Install required NuGet packages
```
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
dotnet add package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer
```

---

## Configuration
1. Configure API Versioning
    - Add this before `builder.Services.AddControllers();`
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

---

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

---

## Mark deprecated
1. Mark API version as deprecated (Controller level)
``` cs
[ApiController]
[ApiVersion("1.0", Deprecated = true)]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/account")]
public sealed class AccountController : ControllerBase
{
    ...
}
```
- v1 still works
- v1 is marked as deprecated
- v2 is the current version

2. Map actions explicitly (recommended)
``` cs
[HttpPost("login")]
[MapToApiVersion("1.0")]
public IActionResult LoginV1(LoginModel model)
{
    // old behavior
}

[HttpPost("login")]
[MapToApiVersion("2.0")]
public IActionResult LoginV2(LoginModel model)
{
    // new behavior
}

```

3. Enable API version reporting
``` cs
options.ReportApiVersions = true;
```
- Resulting response headers for v1 requests
```
api-supported-versions: 1.0, 2.0
api-deprecated-versions: 1.0
```

4. Optional: Add deprecation warning header (recommended)
- Add a filter
``` cs
public sealed class ApiDeprecationHeaderFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }

    public void OnActionExecuted(ActionExecutedContext context)
    {
        var apiVersion = context.HttpContext.GetRequestedApiVersion();

        if (apiVersion?.MajorVersion == 1)
        {
            context.HttpContext.Response.Headers.Add(
                "Warning",
                "299 - \"API v1 is deprecated and will be removed in a future release\""
            );
        }
    }
}
```

- Register it globally
```
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ApiDeprecationHeaderFilter>();
});
```

5. Optional: Swagger deprecation (if you use Swagger)
``` cs
[ApiVersion("1.0", Deprecated = true)]
```

6. Optional: Hard-block v1 after grace period
- Option A — remove v1 mapping: the response=> 404 Unsupported API Version
``` cs
// Remove ApiVersion("1.0")
[ApiVersion("2.0")]
```

- Option B — explicit rejection (custom message)
``` cs
[HttpPost("login")]
[MapToApiVersion("1.0")]
public IActionResult LoginV1()
{
    return StatusCode(
        StatusCodes.Status410Gone,
        "API v1 has been removed. Please upgrade to v2."
    );
}
```

> Recommended deprecation timeline
> 
| Phase   | Action |
|:--------|:------:|
| Phase 1 | Mark deprecated (`Deprecated = true`) |
| Phase 2 | Add warning headers |
| Phase 3 | Notify consumers |
| Phase 4 | Block with **410 Gone** |
| Phase 5 | Remove code |

