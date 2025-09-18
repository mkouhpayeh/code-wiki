# Filters

## Filter Types
- Authorization filters 
- Resources filters (run right after the authorization filters and useful for caching and performance)
- Action filters (run right before and after the action method)
- Exception filters (to handle the exceptions before the response body is populated)
- Result filters (run before and after the execution of the action methods result)


## Create Exp filter
- In the "Exceptions" Folder create another Folder named "Filters":
  
``` cs Title="CustomExceptionFilter.cs"
[AttributeUsage(AttributeTargets.Class | AttributeRargets.Method)] 
public class CustomExceptionFilter : ExceptionFilterAttribute
{
  public override void OnException(ExceptionContext context)
  {
    context.HttpContext.Response.ContentType = "application/json";
    var statusCode = HttpStatusCode.InternalServerError;
    if(context.Exception is StudentNameException)
    {
      statusCode = HttpStatusCode.NotFound;
    }
    var expString = new ErrorResponseData()
    {
      StatusCode = (int)stausCode,
      Message = context.Exception.Message,
      Path = context.Exception.StackTrace
    };
    context.Result = new JsonResult(expString);
  }
}
```

## Use Exp filter
- Just use as an Attribute on top of each method in the controller for example.
