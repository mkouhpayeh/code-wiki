# Authentication
- Authentication is the process of validating the identity of a registered user who is accessing a service or an application.
- Authorization is the process of validating the authenticated user if the user has permissions to access a certain service or an application.

---

## Cookie-based Auth
- In a typical cookie-based authentication flow, a browser communicates directly with a server.
    1. The client sends authentication credentials (e.g. username and password) to the server, usually via an endpoint such as /authenticate.
    2. The server validates the credentials. If they are valid, it creates a server-side session, typically stored in memory or a session store.
    3. The server returns a sessionId to the client. This identifier is stored in a browser cookie and is automatically included in subsequent requests.
    4. For every follow-up request, the browser sends the cookie containing the sessionId. The server uses this value to locate the corresponding session and authenticate the user.
- This approach is called cookie-based authentication because authentication relies on a session identifier stored in a cookie.
- Limitations:
    1. Scalability: Each authenticated user requires a server-side session. At large scale (e.g. millions of concurrent users), maintaining and synchronizing these sessions can become expensive and impact performance.
    2. Security Risk: If authentication cookies are compromised (e.g. via XSS or insecure transport), the attacker gains access to the sessionId and can impersonate the user until the session expires or is revoked.

---

## Token-based Auth
- In token-based authentication, the client and server interact without creating server-side sessions.
    1. The client sends authentication credentials (such as username and password) to the server.
    2. The server validates the credentials. If they are valid, the server does not create a session. Instead, it generates a token.
    3. The token is a cryptographically signed (and sometimes encrypted) string that contains enough information for the server to identify the user and validate the request.
    4. The token is returned to the client and stored in the browser (for example, in memory or secure storage).
    5. For subsequent requests, the client sends the token in the Authorization header using the Bearer scheme:
        ``` cs
        Authorization: Bearer <token>
        ```
        
    6. The server validates the token on each request and uses the information inside it to authenticate the user.

- Token Lifetime
    - Tokens have a limited expiration time.
    - Access tokens are typically short-lived (often 5–15 minutes).
    - If an access token is compromised, its impact is limited by its short lifetime.

- Access Tokens and Refresh Tokens
Token-based systems usually use two types of tokens:
    - Access Token: A short-lived token used to authenticate API requests.
    - Refresh Token: A long-lived token used to obtain a new access token once the current one expires.
      
---

## Json Web Token
- The [JWT](https://jwt.io) is an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.
- It has three parts:
    1. Header: Type of token, signing algorithm
    2. Payload: Claims(Registered, Public, Private)
    3. Signature: Signature(Secret)

---

## Setup EF & JWT
- Install NuGet packages:
    - Microsoft.EntityFrameworkCore
    - Microsoft.EntityFrameworkCore.SqlServer
    - Microsoft.AspNetCore.Identity.EntityFrameworkCore
    - Microsoft.EntityFrameworkCore.Tools
    - Microsoft.AspNetCore.Authentication.JwtBearer

``` cs title="appsettings.json"
"JwtSettings":{
  "SecretKey": "xxxxxxxxxx",
  "Issuer": "https://localhost:5002/",
  "Audience": "user",
  "AccessTokenMinutes" : "10",
  "RefreshTokenDays" : "180"
}
```

``` cs title="Settings POCO"
public sealed class JwtSettings
{
    public string SecretKey { get; set; } = null!;
    public string Issuer { get; set; } = null!;
    public string Audience { get; set; } = null!;
    public int AccessTokenMinutes { get; set; }; 
    public int RefreshTokenDays { get; set; };
}
```

``` cs title="Program.cs"
var builder = WebApplication.CreateBuilder(args);
var configuration = new ConfigurationBuilder()
    .SetBasePath(Environment.CurrentDirectory)
    .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true) // lowest priority
    .AddUserSecrets<Program>(optional: true, reloadOnChange: true)         // overrides json
    .AddEnvironmentVariables()                                            // highest priority
    .Build();

builder.Services.Configure<JwtSettings>(configuration.GetSection("JwtSettings"));
var jwt = configuration.GetSection("JwtSettings").Get<JwtSettings>()!;

builder.Services.AddDbContext<AppDbContext>(opt =>
    opt.UseSqlServer(configuration.GetConnectionString("DBConnection")));

builder.Services
    .AddIdentity<User, Role>(options =>
    {
        options.User.RequireUniqueEmail = true;
        options.Password.RequireDigit = true;
        options.Password.RequiredLength = 8;
        options.Password.RequiredUniqueChars = 2;
        options.Password.RequireLowercase = true;
        options.Password.RequireNonAlphanumeric = true;
        options.Password.RequireUppercase = true;
        options.Lockout.AllowedForNewUsers = true;
        options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
        options.Lockout.MaxFailedAccessAttempts = 5;
        options.SignIn.RequireConfirmedEmail = true;
    })
    .AddEntityFrameworkStores<AppDbContext>()
    .AddDefaultTokenProviders();

var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwt.SecretKey));
var tokenValidationParameters = new TokenValidationParameters
{
    ValidateIssuerSigningKey = true,
    IssuerSigningKey = key,

    ValidateIssuer = true,
    ValidIssuer = jwt.Issuer,

    ValidateAudience = true,
    ValidAudience = jwt.Audience,

    ValidateLifetime = true,
    ClockSkew = TimeSpan.Zero
};
builder.Services.AddSingleton(tokenValidationParameters);

builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    options.SaveToken = true;
    options.RequireHttpsMetadata = false; // set true in production
    options.TokenValidationParameters = tokenValidationParameters;
});

var app = builder.Build();

app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

await AppDbInitializer.SeedRoles(app);

app.MapControllers();
app.Run();
```

``` cs title="DBContext & Models"
public class AppDbContext : IdentityDbContext<
    User, Role, long,
    IdentityUserClaim<long>, IdentityUserRole<long>, IdentityUserLogin<long>,
    IdentityRoleClaim<long>, IdentityUserToken<long>>
{
    public DbSet<UserProfile> UserProfiles => Set<UserProfile>();
    public DbSet<RefreshToken> RefreshTokens => Set<RefreshToken>();

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        modelBuilder.HasDefaultSchema("Identity");

        // Rename Identity tables (remove AspNet*)
        builder.Entity<User>().ToTable("User");
        builder.Entity<Role>().ToTable("Role");
        builder.Entity<IdentityUserRole<long>>().ToTable("UserRole");
        builder.Entity<IdentityUserClaim<long>>().ToTable("UserClaim");
        builder.Entity<IdentityUserLogin<long>>().ToTable("UserLogin");
        builder.Entity<IdentityRoleClaim<long>>().ToTable("RoleClaim");
        builder.Entity<IdentityUserToken<long>>().ToTable("UserToken");

        // (Optional) tweak PK/indices names if you want consistent naming, EF will generate defaults otherwise
        // Example:
        // builder.Entity<User>().HasKey(u => u.Id).HasName("PK_Users");

        // Ensure FK columns are bigint where needed—Identity generics <long> already do that.
    }
}

public class User : IdentityUser<long>
{
    public override long Id { get; set; }
    public bool IsApproved { get; set; }
}

public class Role : IdentityRole<long>
{
    public long? ParentRoleId { get; set; }
    public string? Description { get; set; }
    public bool IsDeleted { get; set; } = false;
    public virtual ICollection<IdentityRoleClaim<long>> RoleClaims { get; set; } = new List<IdentityRoleClaim<long>>();
    [ForeignKey(nameof(ParentRoleId))] public virtual Role? ParentRole { get; set; }
}

public class UserProfile
{
    [Key, DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public long Id { get; set; }
    public long UserId { get; set; }
    public string? CompanyName { get; set; }
    public required string Firstname { get; set; }
    public required string Lastname { get; set; }
    public string? PostalCode { get; set; }
    public string? City { get; set; }
    public string? Street { get; set; }
    public string? Description { get; set; }
    public virtual User User { get; set; } = null!;
}

public class RefreshToken
{
    [Key] public long Id { get; set; }
    public string Token { get; set; } = null!;
    public string JwtId { get; set; } = null!;
    public bool IsRevoked { get; set; }
    public DateTime DateAdded { get; set; }
    public DateTime DateExpire { get; set; }

    public long UserId { get; set; }
    [ForeignKey(nameof(UserId))] public User User { get; set; } = null!;
}
```

``` cs title="Controller"
[ApiController]
[Route("[controller]")]
public class AccountController : ControllerBase
{
    private readonly UserManager<User> _userManager;
    private readonly RoleManager<Role> _roleManager;
    private readonly AppDbContext _context;
    private readonly JwtSettings _jwt;
    private readonly TokenValidationParameters _tokenValidationParameters;

    public AccountController(
        AppDbContext context,
        UserManager<User> userManager,
        RoleManager<Role> roleManager,
        IOptions<JwtSettings> jwt,
        TokenValidationParameters tokenValidationParameters)
    {
        _context = context;
        _userManager = userManager;
        _roleManager = roleManager;
        _jwt = jwt.Value;
        _tokenValidationParameters = tokenValidationParameters;
    }

    [HttpPost("RegisterWithPassword")]
    public async Task<IActionResult> RegisterWithPassword([FromBody] RegisterWithPasswordModel model)
    {
        if (!ModelState.IsValid) return BadRequest("Provide all fields");

        var existing = await _userManager.FindByEmailAsync(model.Email);
        if (existing != null) return BadRequest("User exists");

        using var tx = await _context.Database.BeginTransactionAsync();

        var user = new User { Email = model.Email, UserName = model.Email };
        var result = await _userManager.CreateAsync(user, model.Password);

        if (!result.Succeeded)
        {
            await tx.RollbackAsync();
            return Unauthorized("User creation failed");
        }

        await _userManager.AddToRoleAsync(user, "User");

        await _context.UserProfiles.AddAsync(new UserProfile
        {
            UserId = user.Id,
            Firstname = model.Firstname,
            Lastname = model.Lastname
        });
        await _context.SaveChangesAsync();

        // send email confirmation
        await SendConfirmationEmail(user);

        await tx.CommitAsync();
        return Ok("User created. Please confirm your email.");
    }

    [HttpPost("LoginWithPassword")]
    public async Task<IActionResult> LoginWithPassword([FromBody] LoginWithPasswordModel model)
    {
        if (!ModelState.IsValid) return BadRequest("Not valid");

        var user = await _userManager.FindByEmailAsync(model.Email);
        if (user == null) return Unauthorized("User does not exist");
        if (!await _userManager.IsEmailConfirmedAsync(user)) return Unauthorized("Email is not confirmed");

        if (!await _userManager.CheckPasswordAsync(user, model.Password))
            return Unauthorized("Invalid credentials");

        var auth = await GenerateJwtTokenAsync(user, null);
        return Ok(auth);
    }

    [HttpPost("RefreshToken")]
    public async Task<IActionResult> RefreshToken([FromBody] TokenRequestModel model)
    {
        if (!ModelState.IsValid) return BadRequest("Not valid");
        var result = await VerifyAndGenerateTokenAsync(model);
        if (result is null) return Unauthorized("Invalid refresh request");
        return Ok(result);
    }

    private async Task<AuthResultModel?> VerifyAndGenerateTokenAsync(TokenRequestModel model)
    {
        var stored = await _context.RefreshTokens
            .Include(r => r.User)
            .FirstOrDefaultAsync(x => x.Token == model.RefreshToken);

        if (stored is null || stored.IsRevoked || stored.DateExpire <= DateTime.UtcNow)
            return null;

        var handler = new JwtSecurityTokenHandler();
        SecurityToken validated;
        try
        {
            handler.ValidateToken(model.Token, _tokenValidationParameters, out validated);
            // If token is still valid, no need to refresh (optional behavior)
            return null;
        }
        catch (SecurityTokenExpiredException)
        {
            // Check JTI match
            var expiredToken = handler.ReadJwtToken(model.Token);
            var jti = expiredToken.Claims.FirstOrDefault(c => c.Type == JwtRegisteredClaimNames.Jti)?.Value;
            if (jti != stored.JwtId) return null;

            return await GenerateJwtTokenAsync(stored.User, stored /* for reuse or rotation */);
        }
        catch
        {
            return null;
        }
    }

    private async Task<AuthResultModel> GenerateJwtTokenAsync(User user, RefreshToken? existingRefresh)
    {
        var claims = new List<Claim>
        {
            new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
            new Claim(ClaimTypes.Name, user.UserName ?? user.Email ?? user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Sub, user.Email ?? user.UserName ?? user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email ?? string.Empty),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        var roles = await _userManager.GetRolesAsync(user);
        foreach (var role in roles)
            claims.Add(new Claim(ClaimTypes.Role, role));

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwt.SecretKey));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        var token = new JwtSecurityToken(
            issuer: _jwt.Issuer,
            audience: _jwt.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwt.AccessTokenMinutes),
            signingCredentials: creds);

        var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

        RefreshToken refresh;
        if (existingRefresh != null && existingRefresh.DateExpire > DateTime.UtcNow && !existingRefresh.IsRevoked)
        {
            refresh = existingRefresh; // or rotate if you prefer
        }
        else
        {
            refresh = new RefreshToken
            {
                JwtId = token.Id, // this is the JTI (matches token.Id)
                IsRevoked = false,
                UserId = user.Id,
                DateAdded = DateTime.UtcNow,
                DateExpire = DateTime.UtcNow.AddDays(_jwt.RefreshTokenDays),
                Token = $"{Guid.NewGuid()}-{Guid.NewGuid()}"
            };
            _context.RefreshTokens.Add(refresh);
            await _context.SaveChangesAsync();
        }

        return new AuthResultModel
        {
            Token = tokenString,
            RefreshToken = refresh,
            ExpiresAt = token.ValidTo
        };
    }

     private async Task SendConfirmationEmail(User user)
     {
        // Generate an email confirmation token for the user
        var code = await _userManager.GenerateEmailConfirmationTokenAsync(user);
      
        string expireToken = TokenManager.GenerateToken(Convert.ToInt32(_config.GetSection("TimeSettings:AccountActivationTime").Value));
      
        // Build the callback URL with the confirmation code
        //var callbackUrl = Url.Action("ConfirmEmail", "Account", new { userId = user.Id.ToString(), code = code, token = expireToken }, protocol: HttpContext.Request.Scheme);
      
        var callbackUrl = $"{_config.GetValue<string>("Production:ProductionUrl")}/Account/ConfirmEmail?userId={user.Id}&code={Uri.EscapeDataString(code)}&token={Uri.EscapeDataString(expireToken)}";
      
        // Compose the email content with the confirmation link
        await _emailSender.SendEmailAsync(
            user.Email,
            "Aktivieren Sie Ihr Konto",
            GenerateEmailContent("EmailConfirmationTemplate", "EmailAddress", user.Email, "CallbackUrl", callbackUrl));
     }

      [HttpGet]
      [AllowAnonymous]
      [Route("ConfirmEmail")]
      public async Task<IActionResult> ConfirmEmail(string userId, string code, string token)
      {
        // Check for valid input parameters
        if (string.IsNullOrEmpty(userId) || string.IsNullOrEmpty(code) || string.IsNullOrEmpty(token))
        {
            return BadRequest("Invalid request! Make a contact with the administrator.");
        }
        
        // Find the user based on the provided user ID
        var user = await _userManager.FindByIdAsync(userId);
        
        // Check if the user exists
        if (user == null)
        {
            //return Unauthorized("Invalid request!");
        }
        
        if (TokenManager.IsTokenValid(token))
        {
            // Confirm the user's email with the provided confirmation code
            var result = await _userManager.ConfirmEmailAsync(user, code);
        
            // Check if the email confirmation was successful
            if (result.Succeeded)
            {
                return Ok("Your email has been successfully confirmed. Once your account be activated by Helmsauer's team, you'll be able to log in.");
            }
            return Unauthorized("Email confirmation failed!!! Make a contact with the administrator.");
        }
      }

    private string GenerateEmailContent(string template, params object[] data)
    {
        try
        {
            // Read the HTML content from the external template file
            string htmlContent = System.IO.File.ReadAllText($"Models/DataModels/{template}.html");
    
            //in html file use {EmailAddress} as placeholder
    
            if (data != null)
            {
                for (int i = 0; i < data.Length; i += 2)
                {
                    if (i + 1 < data.Length && data[i] != null)
                    {
                        string placeholder = $"{{{data[i]}}}";
                        htmlContent = htmlContent.Replace(placeholder, data[i + 1].ToString());
                    }
                }
            }
    
            return htmlContent;
        }
        catch
        {
            // Propagate any exceptions that occur during template file reading
            throw;
        }
      }
}

```

``` cs title="Role seeding"
public static class AppDbInitializer
{
    public static async Task SeedRoles(IApplicationBuilder appBuilder)
    {
        using var scope = appBuilder.ApplicationServices.CreateScope();
        var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<Role>>();

        async Task EnsureRole(string name)
        {
            if (!await roleManager.RoleExistsAsync(name))
                await roleManager.CreateAsync(new Role { Name = name });
        }

        await EnsureRole("Admin");
        await EnsureRole("User");
    }
}
```

