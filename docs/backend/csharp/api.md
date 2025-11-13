# API (Application Programming Interface)

## REST API Settings
``` cs title="program.cs"
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();
```

## Controller Attributes
``` cs title="Controllers"
 [ApiController]
 [Route("[controller]")]
```
``` cs title="Actions"
 [Authorize]
 [AllowAnonymous]
 [HttpGet]
 [HttpPost]
 [Route("Index")]
 [ActionName("Confirm")]
```
``` cs title="Preventing a Public Method from Being Invoked"
[NonAction]
```


