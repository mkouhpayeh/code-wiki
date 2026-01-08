# API Server
**Application Programming Interface** 

---

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

---

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

---

## API Definition

- [Entity framework settings](../ef.md)

- Prepare Response structure
    ``` cs title="ResponseModel.cs"
    public class ResponseModel
    {
        public string Message { get; set; }
        public ResponseStatusEnum Status { get; set; }
        public object? Data { get; set; }
    }
    ```
    ``` cs title="ResponseStatusEnum.cs"
    public enum ResponseStatusEnum
    {
        Success = 0,
        NotFound = 1,
        DbException = 2,
        InvalidModelState = 3,
        DuplicateValue = 4,
        Failed = 5,
        TokenExpired = 6,
        InvalidToken = 7,
        Lockedout = 8,
        Unknown = 9,
        Exception = 10,
        Error = 11,
        Unauthorized = 12,
        IsNotAllowed = 13
    }
    ```

- Controllers    
    ``` cs title="ServicesController.cs"
    [HttpGet]
    public async Task<IActionResult> ReadItems()
    {
        try
        {
            var items = await _context.Items
               .IgnoreQueryFilters()
               .ToListAsync();
    
            return Ok(new ResponseModel<List<Item>> { Message = "Items found successfully!", Status = ResponseStatusEnum.Success, Data = items });
        }
        catch (Exception ex)
        {
            // Log
            return StatusCode(500, new ResponseModel<List<Item>> { Message = "Get Items failed! ", Status = ResponseStatusEnum.Exception, Data = null });
        }
    }
    
    ```

---
