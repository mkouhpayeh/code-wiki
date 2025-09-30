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
var isAuth = user.Identity?.IsAuthenticated == true; // "Negotiate" or "NTLM"
User.Identity?.AuthenticationType; 
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

9. Setup IIS 
   0) Prerequisites (once) <br/>
      * Server is domain-joined.
      * DNS host name clients will use exists (e.g. api.contoso.com) and resolves to the server.
      * TLS cert for that host name is installed in Local Computer → Personal → Certificates.
      * Install .NET Hosting Bundle (same major/minor as your app) on the server.
      * In Server Manager → Add Roles & Features → Web Server (IIS):
      * Under Web Server → Security: check Windows Authentication.
      * (Optional) URL Authorization, Logging, etc.
    1) Publish & create the site <br/>
    2) In IIS Manager: <br/>
       * Application Pools → Add
          *  Name: MyApiPool
          *  .NET CLR: No Managed Code
          *  Managed pipeline: Integrated
       *  (Choose identity now or later—see §3.)
       *  Sites → Add Website…
          * Site name: MyApi
          * Physical path: C:\Sites\MyApi
          * Binding:
             * Type: https
             * Host name: api.contoso.com
             * SSL certificate: pick your cert
         * Assign Application Pool → MyApiPool.
      3) Enable Windows auth in IIS <br/>
         * Select the MyApi site → Authentication:
            * Windows Authentication: Enabled
            * Anonymous Authentication: Disabled
         * Still in Windows Authentication → Providers…:
            * Ensure Negotiate is first, NTLM second.
         * Advanced Settings…:
            * Enable Kernel-mode authentication: True (default)
            * Extended Protection: Off (while testing; you’ll re-enable later)
            * UseAppPoolCredentials:
               * True if you’ll run the app pool as a domain service account (option B below).
               * False if using the machine account (option A).
      4) Choose app-pool identity & set SPN (Kerberos) <br/>
      > Kerberos needs an HTTP SPN for the exact DNS name clients use.
      * Option A — Use the server’s computer account (simple)
         * Leave app pool identity as ApplicationPoolIdentity.
         * Register SPN to the computer account:
         ``` shell
         # Run as Domain Admin
         setspn -S HTTP/api.contoso.com CONTOSO\SERVERNAME$
         ```
         * Keep UseAppPoolCredentials = False.
      * Option B — Use a domain service account (recommended for farms)
         * Create CONTOSO\svc-webapi, grant it Log on as a service (GPO or Local Security Policy).
         * Set app pool identity to Custom account → CONTOSO\svc-webapi.
         * Register SPN to that account:
         ``` shell title="powershell"
         setspn -S HTTP/api.contoso.com CONTOSO\svc-webapi
         ```
         * Set UseAppPoolCredentials = True (Windows Authentication → Advanced Settings).
          ``` shell title="powershell"
         setspn -Q HTTP/api.contoso.com
         ```
         If duplicates exist, Kerberos fails → browser prompts.
      5) Configure the ASP.NET Core app <br/>
      ``` cs title="Program.cs"
      using Microsoft.AspNetCore.Server.IISIntegration;

      var builder = WebApplication.CreateBuilder(args);
      
      // Trust IIS’s Windows auth (Integrated Pipeline)
      builder.Services.AddAuthentication(IISDefaults.AuthenticationScheme);
      builder.Services.AddAuthorization(o => o.FallbackPolicy = o.DefaultPolicy);
      
      builder.Services.AddControllers();
      
      var app = builder.Build();
      app.UseRouting();
      app.UseAuthentication();
      app.UseAuthorization();
      app.MapControllers();
      app.Run();
      ```
      (Optional) web.config hardening in your site root <br />
      ``` xml title="web.config"
      <configuration>
        <system.webServer>
          <security>
            <authentication>
              <windowsAuthentication enabled="true" useKernelMode="true" />
              <anonymousAuthentication enabled="false" />
            </authentication>
          </security>
          <handlers>
            <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified"/>
          </handlers>
          <aspNetCore processPath="dotnet" arguments="MyApi.dll" stdoutLogEnabled="false" hostingModel="InProcess" />
        </system.webServer>
      </configuration>
      ```
      6) Client/browser settings (for silent SSO) <br/>
         * Access the site via https://api.contoso.com (not IP, not mismatched alias).
         * On domain PCs, add https://api.contoso.com to Local Intranet zone:
            *  Internet Options → Security → Local intranet → Sites → Advanced → Add.
            *  (Chrome/Edge use these settings.)
         *  For Firefox: about:config → set
            *  network.automatic-ntlm-auth.trusted-uris = contoso.com
         > Non-domain or off-network clients will see a credential prompt—that’s expected.
       7) Test Kerberos is actually used <br/>
          * From a domain PC:
            ``` shell title="powershell"
            Invoke-WebRequest https://api.contoso.com/api/secure -UseDefaultCredentials
            klist | Select-String HTTP/api.contoso.com
            ```
           * In your API (temporary endpoint), log:
             ```
             User.Identity?.Name;             // CONTOSO\jdoe
             User.Identity?.AuthenticationType; // "Negotiate" (Kerberos) or "NTLM"
             ```
        8) If you have a separate frontend origin, enable CORS with credentials in the API:
            ``` cs title="Program.cs"
            builder.Services.AddCors(o => o.AddPolicy("FrontEnd", p =>
                p.WithOrigins("https://app.contoso.com")
                 .AllowAnyHeader().AllowAnyMethod().AllowCredentials()));
   
            app.UseCors("FrontEnd");
            ```
   
* Info
With Windows/Negotiate, the OS validates the user’s ticket/token. If someone creates a local account called DOMAIN\jdoe, they still can’t impersonate your AD user unless they can get a valid Kerberos/NTLM token from your domain controller. Still, you should enforce that only users from your AD (or a trusted one) can call the API.

Here’s the secure, practical setup.
1. Don’t authorize by username strings. User.Identity.Name (e.g. TEST1\jdoe) tells you the domain, but treat it as display info only. For authorization, rely on SIDs / AD group membership, not raw names.
* Check membership by group SID (safe & rename-proof). Create (or reuse) an AD group like APP_MyApi_Users in your domain and put allowed users/groups there. Then enforce membership by SID:
``` cs title="Program.cs"
using System.Security.Principal;
using Microsoft.AspNetCore.Authorization;

builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AppUsersOnly", policy =>
        policy.RequireAssertion(ctx =>
        {
            if (ctx.User.Identity is WindowsIdentity wi && wi.Groups is not null)
            {
                // SID of your AD group, not its name
                var requiredSid = new SecurityIdentifier("S-1-5-21-xxxxxxxxx-xxxxxxxxx-xxxxxxxxx-12345");
                return wi.Groups.Any(g => g.Equals(requiredSid));
            }
            return false;
        }));
});
```
``` cs title="Controller"
[Authorize(Policy = "AppUsersOnly")]
[ApiController]
[Route("[controller]")]
public class SecureController : ControllerBase
{
    [HttpGet("whoami")]
    public IActionResult WhoAmI() => Ok(User.Identity?.Name); // e.g. "TEST1\\jdoe"
}
```
> Get your group’s SID once (PowerShell):
``` shell
Get-ADGroup APP_MyApi_Users | Select -ExpandProperty SID
```

2. Prefer Kerberos, avoid NTLM where possible
Kerberos gives you mutual auth and SPN checks (harder to spoof/relay). To steer things to Kerberos and block NTLM:
* Register the SPN for your service (e.g. HTTP/api.contoso.com) on the app pool identity / service account.
* Make clients access the API via the SPN name (https://api.contoso.com, not a raw IP).
* Disable NTLM via Group Policy (server and/or domain policy) or in IIS (remove “NTLM” provider; keep “Negotiate”).
* Put the site in browsers’ Local Intranet zone so they send tickets automatically.
  
3. Enable TLS and Extended Protection (IIS)
* Use HTTPS only.
* In IIS, turn on Extended Protection for Authentication to bind credentials to the TLS channel (mitigates relay).
* Consider requiring Channel Binding Tokens if your environment supports it.

!!! note "TL;DR"
      Authentication: let Negotiate/OS validate the token (Kerberos preferred).
      Authorization: never by raw names; use membership in your AD group’s SID.
      Constrain domain: check the DOMAIN\user part if needed, but the group SID is the real gate.
      Harden: Kerberos + SPN, disable NTLM, HTTPS, Extended Protection.

   
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
