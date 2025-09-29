# OAuth and OpenID Connect

## Google Ex Auth
1. [Open Google Cloud Dashboard](https://console.cloud.google.com/)
2. Create New Project > APIs and services 
3. Select OAuth consent screen to Configure Google auth platform > (Name (do not contain "google" word), Support Email, External option)
4. Create OAuth client ID > (Application Type, Name, Authorised redirect URIs "https://localhost:5002/signin-google")
5. Install package Microsoft.AspNetCore.Authentication.Google
6. ``` cs title="Program.cs"
     builder.Services.AddAuthentication()
       .AddGoogle(options =>
        {
            options.ClientId = configuration.GetSection("ExAuthentication:Google:ClientId").Value ?? string.Empty;
            options.ClientSecret = configuration.GetSection("ExAuthentication:Google:ClientSecret").Value ?? string.Empty;
            options.SignInScheme = IdentityConstants.ExternalScheme;
            options.CallbackPath = "/signin-google";
        });
   ```
