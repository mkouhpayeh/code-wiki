# HTTP Requests

## Send Request
Use `HttpClient` class to be able to send get or post http requests.

``` cs title="program.cs"
builder.Services.AddScoped<HttpClient>();
```

``` cs
public class Service
{
  private readonly HttpClient _httpClient;

  public Service(IHttpClientFactory httpClientFactory)
  {
    _httpClient = httpClientFactory.CreateClient("UserApiClient");
  }
}
```


### Get Request

#### Get with QueryString
``` cs
 var response = await _httpClient.GetAsync($"{this._apiBaseUrl}/Account/ReadUserProfile?email={email}");
```

#### Get with Model
Since GET requests don't have a body, the model's properties must be included in the URL as key-value pairs.

``` cs
using System.Web; // Add a reference to System.Web for HttpUtility

public static string ToQueryString<T>(T obj)
{
    var properties = from p in typeof(T).GetProperties()
                     where p.GetValue(obj, null) != null
                     select $"{HttpUtility.UrlEncode(p.Name)}={HttpUtility.UrlEncode(p.GetValue(obj, null).ToString())}";
    return string.Join("&", properties);
}
```

``` cs
var model = new UserProfileRequest
{
    Email = "example@example.com",
    Name = "John Doe"
};

var queryString = ToQueryString(model);
var response = await _httpClient.GetAsync($"{this._apiBaseUrl}/Account/ReadUserProfile?{queryString}");
```

``` cs
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class AccountController : ControllerBase
{
    [HttpGet("ReadUserProfile")]
    public IActionResult ReadUserProfile([FromQuery] UserProfileRequest model)
    {
        if (model == null)
        {
            return BadRequest("Invalid request model.");
        }
    }
}
```

### Post Request

#### Post with QueryString
``` cs
 var response = await _httpClient.PostAsJsonAsync($"{this._apiBaseUrl}/Account/Verify?email={email}&code={code}", new StringContent(""));
```

#### Post with Model
``` cs
 var response = await _httpClient.PostAsJsonAsync($"{this._apiBaseUrl}/Account/ForgotPassword", new ForgotPasswordModel { Email = email });
```
