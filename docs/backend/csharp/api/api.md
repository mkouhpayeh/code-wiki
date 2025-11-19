# API
**Application Programming Interface** 

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

## API Definition

- [Entity framework settings](./ef.md)

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

    ``` cs title="ServicesController.cs"
    [HttpGet("ReadItems")]
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
    
    [HttpGet("ReadItemByName")]
    public async Task<IActionResult> ReadItemByName([FromQuery] string name)
    {
        try
        {
            var item = await _context.Items
               .IgnoreQueryFilters()
               .FirstOrDefaultAsync(i=> i.Name == name);
    
            return Ok(new ResponseModel<Item> { Message = "Items found successfully!", Status = ResponseStatusEnum.Success, Data = item });
        }
        catch (Exception ex)
        {
            // Log
            return StatusCode(500, new ResponseModel<Item> { Message = "Get Items failed! ", Status = ResponseStatusEnum.Exception, Data = item });
        }
    }
    
    [HttpPost("UpdateItem")]
    public async Task<IActionResult> UpdateItem([FromBody] Item model)
    {
        try
        {
            if (!ModelState.IsValid)
            {
                // Log
                return BadRequest(new ResponseModel { Message = "Model state is not valid", Status = ResponseStatusEnum.InvalidModelState });
            }
    
                var item = await _context.Items
                    .IgnoreQueryFilters()
                    .FirstOrDefaultAsync(i => i.Id == model.Id);
    
            if (customer == null)
            {
                // with returned value you can find the existings item is Active or Not
                return StatusCode(404, new ResponseModel() { Message = "Item not found!", Status = ResponseStatusEnum.NotFound, Data = item });
            }
        
            _context.Entry(item).CurrentValues.SetValues(model);
            await _context.SaveChangesAsync();
        
            return Ok(new ResponseModel<Item> { Message = "Item updated successfully!", Status = ResponseStatusEnum.Success, Data = item });
        }
        
        catch (Exception ex)
        {
            // Log
            return StatusCode(500, new ResponseModel<Item> { Message = "Update item failed! ", Status = ResponseStatusEnum.Exception, Data = item });
        }
    }
    ```

## API Usage
[API Client](./api-client.md)

## API Architecture
[API Structure](./api-arch.md)
