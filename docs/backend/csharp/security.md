# Security

## OWASP

- [**Open Web Application Security Project**](https://owasp.org/Top10)

## Cross-Site Scripting (XSS) Defence

### XSS JS
- If we are outputting something on a razor page in a razor view as part of ASP.NET Core MVC in a blazer control with the **@** character, the data will be automatically HTML escaped.
- So HTML special characters, opening angular bracket, closing angular bracket, double quotes, single quote, and the ampersand character, they are properly escaped. 
- The same functionality is also available using the **@HtmlEncode** method. That's part of system.web.httpUtility.
- Do not use @Html.Raw("<script>alert(hello)</script>")
- <script> jQuery("h1.page-title").html('Search Result: @ViewBag.SearchTerm'); </script>. But the "\" character does not escape. So the js script will run, if we use '\x3c' instead of '<' and '\x3e' instead of '>' => "\x3cscript\x3ealert(hello)\x3c/script\x3e".
- So we should use **@System.Web.HttpUtility.JavaScriptStringEncode()** in js scripts. => scapes all "\" characters.
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
