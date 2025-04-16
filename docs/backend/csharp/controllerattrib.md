# Controller's Attribute

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

## Attributes
``` cs title="Controllers"
 [ApiController]
 [Route("[controller]")]
```
``` cs title="Actions"
 [Authorize]
 [AllowAnonymous]
 [HttpPost]
 [Route("Index")]
 [ActionName("Confirm")]
```
``` cs title="Preventing a Public Method from Being Invoked"
[NonAction]
```


