# Authentication
- Authentication is the process of validating the identity of a registered user who is accessing a service or an application.
- Authorization is the process of validating the authenticated user if the user has permissions to access a certain service or an application.

## Cookie-based Auth
- You typically have a browser and a server. And if you want to be authenticated, you'd send the username and the password to the server. Let's say by using an API endpoint /authenticate.
- Then the server is going to check if the credentials are correct. And if the credentials are correct, then the server is going to create a session in the server memory.
- Then it is going to return to the user that sessionId. This sessionId gets stored in a cookie in the browser. tion type cookie-based authentication because the sessionId gets stored in a cookie.
- Then next, if you want to get some data from the server, you'd pass the sessionId as part of the request as well.
- There was two problems:
    1. The first problem is that if let's say millions of users try to access your app at the same time, then the server is going to create millions of sessions, which is going to overload your server.
    2. The second problem is that if your cookies get stolen from the browser, the sessionIds are stolen as well.
  
## Token-based Auth
- Here is the same way we have a browser and a server, if you want to be authenticated, you would send the username and password, and they server would check if these credentials are valid.
- If the credentials are valid, the server will not create a session, but instead it is going to generate a token.
- This token is just an encrypted string, which has enough valuable information for the server to find out the identity of a user.
- After the token is returned to the browser, the token will be stored in the browser, and on the next request when you want to get some data from the server, you'd use the authorization Bearer and pass the token value.
- The tokens have an expiration time. So even if your token gets stolen, it will expire after typically 5 to 10 minutes, because that is the standard token lifetime.
- We have the short lived tokens, like the access token, or the token that gets returned from the server.
- But we also have long lived tokens. We also call them refresh tokens. So once the token is expired, we use our refresh token to generate a new token.

## Json Web Token
- The [JWT](https://jwt.io) is an open standard that defines a compact and self-contained way for securely transmitting information between parties as a JSON object.
- It has three parts:
    1. Header: Type of token, signing algorithm
    2. Payload: Claims(Registered, Public, Private)
    3. Signature: Signature(Secret)

## Setup EF
``` cs title="AppDbContext.cs - Normal Use"
public class AppDbContext : DbContext
{
  public AppDbContext (DbContextOptions<AppDbContext> options):base(options)
  {
    
  }
}
```

Use Identity Framework:
- Install NuGet packages:
    - Microsoft.EntityFrameworkCore
    - Microsoft.EntityFrameworkCore.SqlServer
    - Microsoft.AspNetCore.Identity.EntityFrameworkCore
    - Microsoft.EntityFrameworkCore.Tools
      
``` cs title="Program.cs"
var configuration = new ConfigurationBuilder()
        .SetBasePath(Environment.CurrentDirectory)
        .AddJsonFile("appsettings.json", optional: true, reloadOnChange: true)
        .AddUserSecrets<Program>(optional: true, reloadOnChange: true)
        .AddEnvironmentVariables()
            .Build();

builder.Services.AddDbContext<AppDbContext>(options =>
{
  options.UseSqlServer(configuration.GetConnectionString("DBConnection"));
});
```

``` cs title="Models/"
public class User : IdentityUser<long>
{
  [Key]
  [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
  public override long Id { get; set; }

  public bool IsApproved { get; set; }
}

public class Role : IdentityRole<long>
{
    public long? ParentRoleId {  get; set; }
    public string? Description { get; set; }
    public bool IsDeleted { get; set; } = false;

    public virtual ICollection<IdentityRoleClaim<long>> RoleClaims { get; set; }

    [ForeignKey("ParentRoleId")]
    public virtual Role ParentRole { get; set; }

}

public class UserProfile
{
    [Key]
    [DatabaseGenerated(DatabaseGeneratedOption.Identity)]
    public long Id { get; set; }
    public long UserId { get; set; }
    public string? CompanyName { get; set; }
    public required string Firstname { get; set; }
    public required string Lastname { get; set; }
    public string? PostalCode { get; set; }
    public  string? City { get; set; }
    public  string? Street { get; set; }
    public string? Description { get; set; }

    public virtual User User { get; set; } = null!;
}

public class AppDbInitializer
{
    public static async Task SeedRoles( IApplicationBuilder appBuilder)
    {
        using (var serviceScope = appBuilder.ApplicationServices.CreateScope())
        {
            var roleManager = serviceScope.ServiceProvider.GetRequiredService<RoleManager<Role>>();
            if(!await roleManager.RoleExistsAsync("Admin")
                await roleManager.createasync(new Role("Admin"));
            if(!await roleManager.RoleExistsAsync("User")
                await roleManager.createasync(new Role("User"));
        }
    }
} 
```

``` cs title="AppDbContext.cs"
public class AppDbContext : IdentityDbContext<User>
{
  public AppDbContext (DbContextOptions<AppDbContext> options): base(options)
  {
    
  }
}
```

``` cs title="Program.cs"
builder.Services.AddIdentity<User, Role>(options =>
{
    options.User.RequireUniqueEmail = true;

    // Password settings
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequiredUniqueChars = 2;
    options.Password.RequireLowercase = true;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequireUppercase = true;

    // Lockout settings
    options.Lockout.AllowedForNewUsers = true;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(5);
    options.Lockout.MaxFailedAccessAttempts = 5;

    // Signin settings
    options.SignIn.RequireConfirmedEmail = true;

})
.AddEntityFrameworkStores<AppDbContext>()
.AddDefaultTokenProviders();


// Seed DB at the end of the file
AppDbInitializer.SeedRoles(app).Waite();
```

## Setup JWT
- Install NuGet packages:
    - Microsoft.AspNetCore.Authentication.JwtBearer
      
``` cs title="appsettings.json"
"JwtSettings":{
  "SecretKey": "xxxxxxxxxx",
  "Issuer": "https://localhost:5002/",
  "Audience": "user"
}
```

``` cs title="Program.cs"
var jwtSettings = configuration.GetSection("JwtSettings"); 
var tokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(jwtSettings["SecretKey"]))

        ValidateIssuer = true,        
        ValidIssuer = jwtSettings["Issuer"],

        ValidateAudience = true,
        ValidAudience = jwtSettings["Audience"],

        ValidateLifetime = true
        ClockSkew = TimeSpan.Zero, // Default is 10 mins
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
    opions.SaveToken=true;
    options.RequireHttpsMetadata = false;
    options.TokenValidationParameters = tokenValidationParameters
})

app.UseRouting();

//After UseRouting()
app.UseAuthentication();
app.UseAuthorization();

```

``` cs title="Models/"
public class RegisterWithPasswordModel
{
    [EmailAddress]
    public required string Email { get; set; }
    public required string Password { get; set; }
    public required string Firstname { get; set; }
    public required string Lastname { get; set; } 
}

public class LoginWithPasswordModel
{
    [EmailAddress]
    public required string Email { get; set; }
    public required string Password { get; set; }
    public bool RememberMe { get; set; }
    public bool lockoutOnFailure { get; set; }
}

public class AuthResultModel
{
    public string Token { get; set; }
    public RefreshToken RefreshToken { get; set; }
    public DateTime ExpiresAt { get; set; }
}

public class RefreshToken
{
  [Key]
  public long Id { get; set; }

  public string Token { get; set; }
  public string JwtId { get; set; }
  public bool IsRevoked { get; set; }
  public DateTime DateAdded { get; set; }
  public DateTime DateExpire { get; set; }

  public string UserId { get; set; }
  [ForeignKey(nameof(UserId))]
  public User User { get; set; }
}

public class TokenRequestModel
{
  [Required]
  public string Token { get; set; }
  [Required]
  public string RefreshToken { get; set; }
}
```

``` cs title="Controllers/"
[ApiController]
[Route("[controller]")]
public class AccountController : ControllerBase
{
  private readonly UserManager<User> _userManager;
  private readonly RoleManager<Role> _roleManager;
  private readonly AppDbContext _context;
  private readonly IConfiguration _config;
  private readonly TokenValidationParameters _tokenValidationParameters;

  public AccountController(AppDbContext context, UserManager<User> userManager, RoleManager<Role> roleManager, IConfiguration config, TokenValidationParameters tokenValidationParameters)
  {
    _context = context;
    _userManager = userManager;
    _roleManager = roleManager;
    _config = config;
    _tokenValidationParameters = tokenValidationParameters;
  }

  [HttpPost]
  [Route("RegisterWithPassword")]
  public async Task<IActionResult> RegisterWithPassword([FromBody] RegisterWithPasswordModel model)
  {
    if (!ModelState.IsValid)
    {
      return BadRequest("Provide all fields");
    }

    var userExists = await _userManager.FindByEmailAsync(model.Email);
    if (userExists != null)
    {
        return BadRequest("User exists");
    }

    using (var transaction = _context.Database.BeginTransaction())
    {
      var user = new User { Email = model.Email, UserName = model.Email };
      var result = await _userManager.CreateAsync(user, model.Password);

      if (result.Succeeded)
      {
        // Send confirmation email
        await SendConfirmationEmail(user);

        // Sign in the user and commit the transaction
        await _signInManager.SignInAsync(user, isPersistent: false);

        await _userManager.AddToRoleAsync(user, "User");

        //Create Profile 
        var userProfile = new UserProfile
        {
            UserId = user.Id,
            Firstname = model.Firstname,
            Lastname = model.Lastname,
            PostalCode = "-",
            CompanyName = "-",
            City = "-",
            Street = "-",
            Description = "-"
        };

        // Add the new profile to the database and save changes
        await _context.UserProfiles.AddAsync(userProfile);
        await _context.SaveChangesAsync();

        await transaction.CommitAsync();
        return Ok("User created");
       }
        await transaction.RollbackAsync();
        return Unauthorized("User creation failed");
    }  
  }

//[Authorize(Roles="Admin,User")]
  [HttpPost]
  [Route("LoginWithPassword")]
  public async Task<IActionResult> LoginWithPassword([FromBody] LoginWithPasswordModel model)
  {
    if (!ModelState.IsValid)
    {
        return BadRequest("Not valid");
    }

    var user = await _userManager.FindByEmailAsync(model.Email);
    
    if (user == null)
    {
        return Unauthorized("User does not exist");
    }

    if (!await _userManager.IsEmailConfirmedAsync(user))
    {
        return Unauthorized("Email is not confirmed");
    }

    var signInResult = await _signInManager.PasswordSignInAsync(model.Email, model.Password, model.RememberMe, lockoutOnFailure: true);

    // Check the result of the login attempt
    if (signInResult.Succeeded || signInResult.RequiresTwoFactor)
    {
      // return required data
      var tokenValue = await GenerateJwtTokenAsync(user, null);
      return Ok(tokenValue);
    }

    if (signInResult.IsLockedOut)
    {
      // return unauthorized  
      return StatusCode(401, "User is lockedout");
    }
  }

  [HttpPost]
  [Route("RefreshToken")]
  public async Task<IActionResult> RefreshToken([FromBody] TokenRequestModel model)
  {
    if (!ModelState.IsValid)
    {
        return BadRequest("Not valid");
    }

    var result = VerifyAndGenerateTokenAsync(model);
    return Ok(result);
  }

  private async Task<AuthResultModel> VerifyAndGenerateTokenAsync(TokenRequestModel model)
  {
    var jwtTokenHandler = new JwtSecurityTokenHandler();
    var storedToken = await _context.RefreshTokens.FirstOrDefaultAsync(x=> x.Token == model.RefreshToken);

    var dbUser = await _userManager.FindByIdAsync(storedToken.UserId);

    try // If token is not valid, an exception is going to be thrown
    {
      var tokenCheckResult = jwtTokenHandler.ValidateToken(model.Token, _tokenValidationParameters, out var validatedToken);
      return await GenerateJwtTokenAsync(dbUser, null);
    }
    catch(SecurityTokenExpiredException ex) 
    {
      if (storedToken.DateExpire >= DateTime.UtcNow)
      {
         return await GenerateJwtTokenAsync(dbUser, storedToken);
      }
      else
      {
          return await GenerateJwtTokenAsync(dbUser, null);
      }
    }
  }

  // Generate Access Token
  private async<AuthResultModel> Task GenerateJwtTokenAsync(User user,RefreshToken refreshToken)
  {
    var authClaims = new List<Claim>()
    {
      new Claim(ClaimTypes.Name, user.Username),
      new Claim(ClaimTypes.NameIdentifier, user.Id),
      new Claim(JwtRegisteredClaimNames.Email, user.Email),
      new Claim(JwtRegisteredClaimNames.Sub, user.Email),
      new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
    };

    var userRoles = await _userManager.GetRolesAsync(user);
    foreach (var userRole in userRoles)
    {
        authClaims.Add(new Claim(ClaimTypes.Role));
    }
    
    var token = new JwtSecurityToken(
      issuer: jwtSettings.Issuer,
      audience: jwtSettings.Audience,
      claims: authClaims,
      expires: DateTime.UtcNow.AddMinutes(jwtSettings.ExpirationInMinutes), // Set token expiration
      signingCredentials: new SigningCredentials(
          new SymmetricSecurityKey(Encoding.ASCII.GetBytes(jwtSettings.SecretKey)),
          SecurityAlgorithms.HmacSha256)
    );
    
    var tokenString = new JwtSecurityTokenHandler().WriteToken(token);

    if(refreshToken != null)
    {
      // Generate a new access token
       var refreshTokenresponse = new AuthResultModel()
       {
          Token = tokenString,
          RefreshToken = refreshToken.Token,
          ExpiresAt = tokenString.ValidTo
      };
      return refreshTokenresponse;
    }
  
    var refreshToken = new RefreshToken()
    {
      JwtId = tokenString.Id,
      IsRevoked = false,
      UserId = user.Id,
      DateAdded = DateTime.UtcNow,
      DateExpire = DateTime.UtcNow.AddMonths(6),
      Token = Guid.NewGuid().ToString()+"-"+Guid.NewGuid().ToString()
    };

    await._context.RefreshTokens.AddAsync(refreshToken);
    await _context.SaveChangesAsync();

    var response = new AuthResultModel()
    {
      Token = tokenString,
      RefreshToken = refreshToken,
      ExpiresAt = tokenString.ValidTo
    };

    return response;
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

``` cs  title="Models/"
public class TokenManager
{
    public static string GenerateToken(int minutesToExpire)
    {
        byte[] tokenBytes = new byte[32];
        using (var rng = new System.Security.Cryptography.RNGCryptoServiceProvider())
        {
            rng.GetBytes(tokenBytes);
        }

        DateTime expirationTime = DateTime.UtcNow.AddMinutes(minutesToExpire);
        string token = Convert.ToBase64String(tokenBytes) + "|" + expirationTime.ToString("yyyy-MM-ddTHH:mm:ss");

        return token;
    }

    public static bool IsTokenValid(string token)
    {
        // Split the token to get the expiration time
        string[] tokenParts = token.Split('|');
        if (tokenParts.Length == 2 && DateTime.TryParse(tokenParts[1], out DateTime expirationTime))
        {
            // Check if the token is not expired
            return DateTime.UtcNow < expirationTime;
        }

        // Invalid token format or expired
        return false;
    }
}
```
