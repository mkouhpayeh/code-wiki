# Authentication
- Authentication is the process of validating the identity of a registered user who is accessing a service or an application.
- Authorization is the process of validating the authenticated user if the user has permissions to access a certain service or an application.

---

## Cookie-based Auth
- You typically have a browser and a server. And if you want to be authenticated, you'd send the username and the password to the server. Let's say by using an API endpoint /authenticate.
- Then the server is going to check if the credentials are correct. And if the credentials are correct, then the server is going to create a session in the server memory.
- Then it is going to return to the user that sessionId. This sessionId gets stored in a cookie in the browser. tion type cookie-based authentication because the sessionId gets stored in a cookie.
- Then next, if you want to get some data from the server, you'd pass the sessionId as part of the request as well.
- There was two problems:
    1. The first problem is that if let's say millions of users try to access your app at the same time, then the server is going to create millions of sessions, which is going to overload your server.
    2. The second problem is that if your cookies get stolen from the browser, the sessionIds are stolen as well.

---

## Token-based Auth
- Here is the same way we have a browser and a server, if you want to be authenticated, you would send the username and password, and they server would check if these credentials are valid.
- If the credentials are valid, the server will not create a session, but instead it is going to generate a token.
- This token is just an encrypted string, which has enough valuable information for the server to find out the identity of a user.
- After the token is returned to the browser, the token will be stored in the browser, and on the next request when you want to get some data from the server, you'd use the authorization Bearer and pass the token value.
- The tokens have an expiration time. So even if your token gets stolen, it will expire after typically 5 to 10 minutes, because that is the standard token lifetime.
- We have the short lived tokens, like the access token, or the token that gets returned from the server.
- But we also have long lived tokens. We also call them refresh tokens. So once the token is expired, we use our refresh token to generate a new token.

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

