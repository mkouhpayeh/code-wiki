# External Auth

## Active Directory
1. Install package Microsoft.AspNetCore.Authentication.Negotiate
2. Configure authentication/authorization
``` cs title="Program.cs"
builder.Services.AddAuthentication(NegotiateDefaults.AuthenticationScheme)
   .AddNegotiate();

builder.Services.AddAuthorization(options =>
{
    options.FallbackPolicy = options.DefaultPolicy;
});

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();
```
3. Now any controller/action with [Authorize] (or all, if you used the fallback policy) will require a valid Windows login. You can read the user with:
``` cs 
var user = HttpContext.User;                // ClaimsPrincipal
var name = user.Identity?.Name;             // "DOMAIN\\username"
var isAuth = user.Identity?.IsAuthenticated == true;
```
4. In Razor components:
``` cs
@inject IHttpContextAccessor HttpContextAccessor

@code{
     private async Task CheckLdapAuthenticationAsync()
     {
         var domainUser = HttpContextAccessor.HttpContext?.User?.Identity?.Name;
          if (domainUser == null)
          {
          }
     }
}
```
5. Development hosting (IIS Express) – enable Windows auth
In Properties/launchSettings.json (IIS Express profile):
``` json
"iisSettings": {
  "windowsAuthentication": true,
  "anonymousAuthentication": false
}
```
Or set this in Visual Studio: Project Properties → Debug → App URL (IIS Express) → Enable Windows Authentication (on) / Anonymous (off).
6. Kestrel/self-host
You can run on Kestrel; negotiation still works. For browsers to SSO without a prompt, the site must be in clients’ Local Intranet zone (IE/Edge) or configured in Chrome/Firefox to allow integrated auth for your domain.
7. Authorize by AD group
``` cs
[Authorize(Roles = @"MYDOMAIN\\Developers")]
[ApiController]
[Route("api/[controller]")]
public class SecureController : ControllerBase
{
    [HttpGet("me")]
    public IActionResult Me() => Ok(User.Identity?.Name);
}
```
Or define a policy in AddAuthorization:
``` cs
options.AddPolicy("ITAdmins", policy =>
    policy.RequireRole(@"MYDOMAIN\\IT Admins"));
```
Then:
``` cs
[Authorize(Policy = "ITAdmins")]
public IActionResult AdminOnly() => Ok("Hi admin!");
```
8. If you really want to keep ASP.NET Core Identity together with Windows auth
``` cs title="Program.cs"
app.Use(async (ctx, next) =>
{
    if (ctx.User?.Identity?.IsAuthenticated == true)
    {
        var sam = ctx.User.Identity.Name; // DOMAIN\\user
        using var scope = ctx.RequestServices.CreateScope();
        var userManager = scope.ServiceProvider.GetRequiredService<UserManager<ApplicationUser>>();
        var user = await userManager.FindByNameAsync(sam);
        if (user is null)
        {
            user = new ApplicationUser { UserName = sam };
            await userManager.CreateAsync(user); // no password
            // attach app-specific claims/roles if needed
        }
    }
    await next();
});
```
!!! note "Common pitfalls"
          Anonymous auth left enabled → everyone gets through. Turn it off (IIS/IIS Express) or enforce [Authorize].
          Order of middleware → UseAuthentication() must come before UseAuthorization().
          Cross-domain prompts → if the browser isn’t configured for integrated auth to your API host, it may prompt for credentials.


## Google Ex Auth
1. [Open Google Cloud Dashboard](https://console.cloud.google.com/)
2. Create New Project > APIs and services 
3. Select OAuth consent screen to Configure Google auth platform > (Name (do not contain "google" word), Support Email, External option)
4. Create OAuth client ID > (Application Type, Name, Authorised redirect URIs "https://localhost:5002/signin-google")
5. Install package Microsoft.AspNetCore.Authentication.Google
   ``` cs title="Program.cs"
     builder.Services.AddAuthentication()
       .AddGoogle(options =>
        {
            options.ClientId = configuration.GetSection("ExAuthentication:Google:ClientId").Value ?? string.Empty;
            options.ClientSecret = configuration.GetSection("ExAuthentication:Google:ClientSecret").Value ?? string.Empty;
            options.SignInScheme = IdentityConstants.ExternalScheme;
            options.CallbackPath = "/signin-google";
        });
   ```
