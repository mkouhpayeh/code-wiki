# Exception Handler
## Create Exp Handler

``` csharp
public class ErrorResponseData
{
  public int StatusCode {get;set}
  public string Message {get;set;}
  public string Path {get;set;}

  public override string ToString()
  {
    return JsonConvert.SerializeObject(this);
  }
}
```

### Exp Handler Extension

``` csharp
public static class ExceptionMiddlewareExtensions
{
  public static void ConfigureBuiltInExceptionHandler(this IApplicationBuilder app)
  {
    app.UseExceptionHandler(appError =>
    {
      appError.Run(async context =>
      {
        var contextFeature = context.Features.Get<IExceptionHandlerFeature>();
        var contextRequest = context.Features.Get<IHttpRequestFeature>();

        context.Response.ContentType = "application/json";

        if (contextFeature != null)
        {
          var errorString = new ErrorResponseData()
            {
              StatusCode = (int)HttpStatusCode.InternalError,
              Message = contextFeature.Error.Message,
              Path = contextRequest.Path
            }.ToString();
          await context.Response.WriteAsync(errorString);
        }
      });
    });
  }
```

### Configure Exp Handler

``` csharp Title="Program.cs"
app.UseAuthorization();
app.ConfigureBuiltInExceptionHandler();
```

## Create Custom Exp Handler

``` csharp
public class CustomExceptionHandler
{
  private readonly RequestDelegate _next;

  public CustomExceptionHandler(RequestDelegate next)
  {
    _next=next;
  }

  public async Task InvokeAsync(HttpContext httpContext)
  {
    try
    {
      await _next(httpContext);
    }
    catch (Exception ex)
    {
      await HandleExceptionAsync(httpContext, ex);
    }
  }

  private Task HandleExceptionAsync(HttpContext httpContext, Exception ex)
  {
    httpContext.Response.ContentType = "application/json";
    var errorMessageString = new ErrorResponseData()
    {
      StatusCode = (int)HttpStatusCode.InternalServerError,
      Message = ex.Message,
      Path = httpContext.Request.Path
    }.ToString();
    return httpContext.Response.WriteAsync(errorMessageString);
  }
}
```

### Custom Exp Handler Extension
- In the ExceptionMiddlewareExtensions.cs class already we have an extension now we will add another one.

``` cs
 public static void ConfigureCustomExceptionHandler(this IApplicationBuilder app)
  {
    app.UseMiddleware<CustomExceptionHandler>();
  }
```
### Configure Custom Exp Handler
``` cs Title="Program.cs"
app.UseAuthorization();
//app.ConfigureBuiltInExceptionHandler();
app.ConfigureCustomExceptionHandler();
```
