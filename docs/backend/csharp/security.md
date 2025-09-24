# Security

## OWASP

- [**Open Web Application Security Project**](https://owasp.org/Top10)

## Common Attacks
### Cross-Site Scripting (XSS) 
- If we are outputting something on a razor page in a razor view as part of ASP.NET Core MVC in a blazer control with the **@** character, the data will be automatically HTML escaped.
- Replacing Html.Raw with @ when outputting data ensures that HTML is encoded, preventing the execution of unintended JavaScript code.
- So HTML special characters, opening angular bracket, closing angular bracket, double quotes, single quote, and the ampersand character, they are properly escaped. 
- The same functionality is also available using the **@HtmlEncode** method. That's part of system.web.httpUtility.
- Do not use @Html.Raw("<script>alert(hello)</script>")
- <script> jQuery("h1.page-title").html('Search Result: @ViewBag.SearchTerm'); </script>. But the "\" character does not escape. So the js script will run, if we use '\x3c' instead of '<' and '\x3e' instead of '>' => "\x3cscript\x3ealert(hello)\x3c/script\x3e".
- So we should use **@System.Web.HttpUtility.JavaScriptStringEncode()** in js scripts. => scapes all "\" characters. Using JavaScriptStringEncode ensures that all JavaScript special characters are properly escaped, preventing code injection vulnerabilities.
- **If you can just avoid using .NET code and binary data with add within JavaScript. Just stick with an HTML.**

### XSS SPA
- Angular has a pretty good security posture. It is also escaping things like the href attribute for links. Use **{{}}** is HTML-escaped, but **[]** is not.
- React **{}** is HTML-escaped and **{{}}** is not.
- Vue.js **{{}}** is HTML-escaped but **v-html=""** and **innetHTML={}** is not.

### Origin Policy
- It is about the Protocol, Host and Domain.
    -  Protocol: http or https
    -  Domain name: https://test.com or https://www.test.com
    -  Port
-  JavaScript has only access to things on the same origin, and the origin is defined by the HTML page that contains the JavaScript code. So if you run an application on one origin and the JavaScript code then tries to access something, for instance, an API on a different origin, then that does not work.
    -  When we use JavaScript to, say, call an API on a different origin, then the browser first contacts that server and asks whether that cross-origin request is in order. The browser does that by sending the Origin HTTP header to the remote server. And the remote server then can, so to speak, green-light a future request from that origin by returning a different HTTP header called Access-Control-Allow-Origin, and the value of that header has to be the same as the value of that Origin header you just saw. And only if that header is returned to the client, the client then knows, okay, JavaScript is allowed to send that HTTP request and then, well, sends the request to the server.
#### Enabling CORS
- [EnableCors(Policy)] attribute
- [DisableCors] attribute
- Configure in Program.cs
``` cs title="Program.cs"
builder.Services.AddCors(options =>
{
  options.AddPolicy(
    "Policy1",
    builder =>
    {
      builder.WithOrigins("https://localhost:5001");
    });
});

app.UseCors("Policy1");
```

### SQL Injection
- Do not send the value of ID as a string
- ADO.net: Do not use string concatanation to write the sql command. (1;Drop table Test1)
    -  Use parametrized queries with **@** and the name as a placeholder.
- Entity Framework
    - Avoid execute raw SQL: **DbContext.Database.SqlQueryRaw()** or **DbContext.Database.ExecuteRawQuery()** or **DbContext.SomeEntity.FromSqlRaw()**
    - Return **IEnumerable** instead of **IQueryable**

### Cross-Site Request Forgery (CSRF === C-Serf)
- CSRF is an attack where malicious actions are executed on behalf of the victim without their consent, leveraging their authenticated session. Using the data saved in sessions.
- But Asp.net Core has built-in Cross-Site request forgery protection. each request has specific Token. (asp-antiforgery="true")  If you have a form where the method is set to post, then also the request forgery token is automatically added.
- **[ValidateAntiForgeryToken]** enables and **[IgnoreAntiforgeryToken]** will disable the feature.

### SameSite Cookies
- Lax: the cookies only sent on a Cross-Sitee request. If it's an HTTP method, like get, that according to the rest specification does not change the state of the application. For post or put or delete, the cookie is not sent.
- Strict:  if we have a Cross-Site request, the cookie is not sent at all.
- None: no protection
``` cs title="Program.cs"
//builder.Services.AddSessions(options => {options.Cookie.SameSite=SameSite.Mode.None; options.Cookie.SecurePolicy= CookieSecurePolicy.SameRequest;});
builder.Services.AddSessions(); //Lax
```

## Encryption
### Symmetric Key
- Symmetric-key encryption means that we can use the **key both for encrypting the data and for decrypting** it. (AES or DES)

### Public Key
- We have a set of keys: a public key and a private key. And the idea is that we could, for instance, use the **public key for encryption and then the private key for decrypting** the data.

## Storing Data
### Secret Manager
``` cs title="Package Manager Console"
dotnet user-secret init
```
- Then in the SolutionExplore, on right click context menu, select Manage User Secrets. The json file stores in the Profile. Only in local machine.

### App Settings
- appsettings.json is a JSON file with settings.
- There is an extra file, appsettings.development.json. The idea is this, the appsettings.json file contains all of the settings that apply globally, no matter on which environment you are running the application, but per environment you can add specific JSON files that then can have extra settings or override global settings. So appsettings.development.json can override settings from the main appsettings.json file so you could have a different log level for instance. for example the appsettings.production.json
- In Project Properties, Debug, General, just click on 'Open debug launch profiles UI'. Then shows the environment settings for development json file.

### Environment Variables
- The option might be that you're using environment variables for those settings and then read out the environment variables and those environment variables are then for instance, created as part of your CI/CD process.

### Data Protection API
- You have an API to decrypt and to encrypt data, and all of the heavy lifting is done by .NET.

### Blazor Protected Browser Storage
- With a Blazor application that runs in the browser, you can access local storage and session storage, and if you are using the interactive server rendering mode, then the data that you store in local storage and session storage can be encrypted, and the server then leverages the encryption and also the decryption.

### Cloud Storage
- Each of the major cloud providers has a mechanism where you can store values and then have an API to retrieve those values. And usually you store the values once and then, you don't really see them, but your application can retrieve them. So in
    - Azure: Key Vault
    - AWS: Secrets Manager
    - Google Cloud: Secret Manager

## Hashing Data
- Hashing means I have a function that cannot be reversed.
- A hash is like a fingerprint of that specific piece of information. 
- In addition, we are using something called a **salt**. A salt makes sure that if we are hashing the same password a couple of times, having a different salt means that the hashes are different.
- There are different algorithms for creating hashes, and a common one is called **PBKDF2**.

## Security Options
- Token-based Authentication:
    - OpenID Connect: OpenID Connect is a framework on top of OAuth and OpenID Connect is working with identity tokens. So who is the user?
    - OAuth:  OAuth is essentially a standard that works with so-called access tokens. Access tokens basically say, "What is a user allowed to access?"
    > UnAuthorized HTTP Status Code: 401
- Securing SPAs (Single Page App) with the BFF (Backends for Frontrnds) pattern: st Friends Forever, but Backends for Frontends. So the idea essentially boils down to this. You have a server component. That server component takes care of token management, so it stores the token. It refreshes the token if that's required. And the communication between client and that token management system is done using cookies because, once again, cookies can be secured.
- Using Identity Provider such as **IdentityServer**, **OpenIddict**, **Azure**

## Secure Configuration
### Cookies
- A client sends an HTTP request to a server, and the server then can start the Cookie process by returning the Set-Cookie HTTP header alongside the HTTP response. Set-Cookie HTTP header sets a Cookie with a name of value and there can also be Metadata like an expiration date. The client then may choose to store the Cookie or not, and on subsequent requests to that server, that Cookie is sent back. But, this time with a different HTTP header name just Cookie. The client is also sending only the name and the value of the Cookie, not the Metadata. That's something that is for client use only.
``` cs
var options = new CookieOptions {
    Secure = true, //HTTPS
    HttpOnly = true, //invisible for JS
    SameSite = SameSiteMode.Lax //Cookie is being sent with a Cross-Site request, but only if you're using an HTTP method 
};
```
