# Anonymous Endpoint

---

## Allow Anonymous Endpoint

``` scharp title="program.cs"
builder.Services.AddAuthorization(options =>
{
    // By default, all incoming requests will be authorized according to the default policy.
    options.FallbackPolicy = options.DefaultPolicy;
    options.AddPolicy("AllowAnonymousPolicy", policy =>
    {
        policy.RequireAssertion(context =>
            context.Resource is HttpRequest request &&
            (request.Path.StartsWithSegments("/Account/ExternalLogin") ||
             request.Path.StartsWithSegments("/Account/ExternalLoginCallback")));
    });
});
```

``` csharp title="Controller"
[HttpGet]
[Route("ExternalLogin")]
[AllowAnonymous]
public IActionResult ExternalLogin(string provider, string returnUrl = null)
```
