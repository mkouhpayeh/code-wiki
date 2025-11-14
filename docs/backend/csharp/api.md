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
public class ResponseModel<T>
{
    public string Message { get; set; }
    public ResponseStatusEnum Status { get; set; }
    public T Data { get; set; }
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

- 

## API Usage
``` cs
// ResponseStatusEnum.cs
public enum ResponseStatusEnum { Success, InvalidModelState, NotFound, Exception, Conflict }

// ResponseModel.cs
public class ResponseModel<T>
{
    public string Message { get; set; }
    public ResponseStatusEnum Status { get; set; }
    public T Data { get; set; }
}

// Paging.cs
public sealed record PagedRequest(
    int Page = 1,
    int PageSize = 20,
    string? SortBy = null,     // e.g. "Name"
    bool Desc = false,
    string? Filter = null      // free-text or DSL, decide the server format
);

public sealed record PagedResult<T>(
    IReadOnlyList<T> Items,
    int Total,
    int Page,
    int PageSize
);

// Example DTOs (repeat per entity or generate)
public sealed class ItemDto { public int Id { get; set; } public string Name { get; set; } public bool IsActive { get; set; } }
public sealed class CustomerDto { public int Id { get; set; } public string Name { get; set; } public string Email { get; set; } }

```

``` cs title="Program.cs"
using Polly;
using Polly.Extensions.Http;
using System.Net;
using System.Net.Http.Headers;
using System.Net.Http.Json;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddRazorPages();

var apiBase = builder.Configuration["Api:BaseUrl"] 
              ?? throw new InvalidOperationException("Missing Api:BaseUrl");

// shared JSON options
builder.Services.ConfigureHttpJsonOptions(o =>
{
    // Keep property names as-is (match API)
    o.SerializerOptions.PropertyNamingPolicy = null;
});

// resilience
static IAsyncPolicy<HttpResponseMessage> RetryPolicy() =>
    HttpPolicyExtensions
        .HandleTransientHttpError()
        .OrResult(r => r.StatusCode == HttpStatusCode.TooManyRequests)
        .WaitAndRetryAsync(3, retry => TimeSpan.FromMilliseconds(200 * (retry + 1)));

builder.Services.AddHttpClient("Api", client =>
{
    client.BaseAddress = new Uri(apiBase);        // e.g. https://localhost:5001/api/v1/
    client.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    // If you use JWT:
    // client.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", "<token>");
})
.AddPolicyHandler(RetryPolicy())
.SetHandlerLifetime(TimeSpan.FromMinutes(5));

// register generic CRUD client
builder.Services.AddScoped<IApiClientFactory, ApiClientFactory>();

var app = builder.Build();
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.MapRazorPages();
app.Run();

```

``` cs title="appsettings.json"
{
  "Api": { "BaseUrl": "https://localhost:5001/api/v1/" }
}
```

``` cs title="CrudClient.cs"
using System.Net;
using System.Net.Http.Json;
using MyApp.Contracts;

public interface ICrudApi<TDto, TId>
{
    Task<ResponseModel<PagedResult<TDto>>> ListAsync(PagedRequest req, CancellationToken ct = default);
    Task<ResponseModel<TDto>> GetAsync(TId id, CancellationToken ct = default);
    Task<ResponseModel<TDto>> CreateAsync(TDto dto, CancellationToken ct = default);
    Task<ResponseModel<TDto>> UpdateAsync(TId id, TDto dto, CancellationToken ct = default);
    Task<ResponseModel<object>> DeleteAsync(TId id, CancellationToken ct = default);
}

// Builds correctly-typed clients per entity on demand
public interface IApiClientFactory
{
    ICrudApi<TDto, TId> Create<TDto, TId>(string entityRoute);
}

public sealed class ApiClientFactory : IApiClientFactory
{
    private readonly IHttpClientFactory _httpFactory;
    public ApiClientFactory(IHttpClientFactory httpFactory) => _httpFactory = httpFactory;
    public ICrudApi<TDto, TId> Create<TDto, TId>(string entityRoute)
        => new CrudApi<TDto, TId>(_httpFactory.CreateClient("Api"), entityRoute.TrimEnd('/') + "/");
}

internal sealed class CrudApi<TDto, TId> : ICrudApi<TDto, TId>
{
    private readonly HttpClient _http;
    private readonly string _route; // e.g. "items/" (already under /api/v1/)
    public CrudApi(HttpClient http, string route) { _http = http; _route = route; }

    public async Task<ResponseModel<PagedResult<TDto>>> ListAsync(PagedRequest req, CancellationToken ct = default)
    {
        var url = $"{_route}?page={req.Page}&pageSize={req.PageSize}"
                + (string.IsNullOrWhiteSpace(req.SortBy) ? "" : $"&sortBy={Uri.EscapeDataString(req.SortBy)}&desc={req.Desc}")
                + (string.IsNullOrWhiteSpace(req.Filter) ? "" : $"&filter={Uri.EscapeDataString(req.Filter)}");

        return await ReadOrThrow<ResponseModel<PagedResult<TDto>>>(await _http.GetAsync(url, ct), ct);
    }

    public async Task<ResponseModel<TDto>> GetAsync(TId id, CancellationToken ct = default)
        => await ReadOrThrow<ResponseModel<TDto>>(await _http.GetAsync($"{_route}{id}", ct), ct);

    public async Task<ResponseModel<TDto>> CreateAsync(TDto dto, CancellationToken ct = default)
        => await ReadOrThrow<ResponseModel<TDto>>(await _http.PostAsJsonAsync(_route, dto, ct), ct);

    public async Task<ResponseModel<TDto>> UpdateAsync(TId id, TDto dto, CancellationToken ct = default)
        => await ReadOrThrow<ResponseModel<TDto>>(await _http.PutAsJsonAsync($"{_route}{id}", dto, ct), ct);

    public async Task<ResponseModel<object>> DeleteAsync(TId id, CancellationToken ct = default)
        => await ReadOrThrow<ResponseModel<object>>(await _http.DeleteAsync($"{_route}{id}", ct), ct);

    private static async Task<T> ReadOrThrow<T>(HttpResponseMessage res, CancellationToken ct)
    {
        // Prefer server’s envelope
        var contentType = res.Content.Headers.ContentType?.MediaType ?? "";
        try
        {
            var obj = await res.Content.ReadFromJsonAsync<T>(cancellationToken: ct);
            if (obj is not null) return obj;
        }
        catch { /* fall through to ProblemDetails */ }

        // If server returned ProblemDetails (RFC7807)
        if (contentType.Contains("application/problem+json"))
        {
            var problem = await res.Content.ReadFromJsonAsync<Microsoft.AspNetCore.Mvc.ProblemDetails>(cancellationToken: ct);
            throw new ApiProblemException(res.StatusCode, problem?.Title ?? "API error", problem?.Detail);
        }

        var raw = await res.Content.ReadAsStringAsync(ct);
        throw new ApiProblemException(res.StatusCode, $"HTTP {(int)res.StatusCode}", raw);
    }
}

public sealed class ApiProblemException : Exception
{
    public HttpStatusCode StatusCode { get; }
    public ApiProblemException(HttpStatusCode statusCode, string title, string? detail = null)
        : base($"{title}{(string.IsNullOrWhiteSpace(detail) ? "" : $": {detail}")}") => StatusCode = statusCode;
}

```

> Use Create<TDto,TId>("items") or Create<CustomerDto,int>("customers") for each entity. No new code per entity required.

- Using the generic client in Razor Pages `Pages/Items/Index.cshtml.cs`

``` cs title="Index.cshtml.cs"
using Microsoft.AspNetCore.Mvc.RazorPages;
using MyApp.Contracts;

public class IndexModel : PageModel
{
    private readonly ICrudApi<ItemDto, int> _items;
    public IndexModel(IApiClientFactory factory) => _items = factory.Create<ItemDto, int>("items");

    public PagedResult<ItemDto> Result { get; private set; } = new(new List<ItemDto>(), 0, 1, 20);
    public string? Q { get; private set; }
    public string? SortBy { get; private set; }
    public bool Desc { get; private set; }

    public async Task OnGetAsync(int page = 1, int pageSize = 20, string? q = null, string? sortBy = null, bool desc = false)
    {
        Q = q; SortBy = sortBy; Desc = desc;
        var req = new PagedRequest(page, pageSize, sortBy, desc, q);
        var resp = await _items.ListAsync(req);
        Result = resp.Data ?? new PagedResult<ItemDto>(Array.Empty<ItemDto>(), 0, page, pageSize);
        // You can use resp.Status/Message for banners
    }
}
```

``` html title="Index.cshtml"
@page
@model IndexModel
@{
    ViewData["Title"] = "Items";
}
<h2>Items</h2>

<form method="get" class="mb-3">
  <input type="text" name="q" value="@Model.Q" placeholder="Search..." />
  <select name="sortBy">
    <option value="">(no sort)</option>
    <option value="Name" selected="@(Model.SortBy=="Name")">Name</option>
    <option value="Id" selected="@(Model.SortBy=="Id")">Id</option>
  </select>
  <label><input type="checkbox" name="desc" value="true" @(Model.Desc ? "checked" : "") /> Desc</label>
  <button type="submit">Apply</button>
</form>

<table class="table">
  <thead><tr><th>Id</th><th>Name</th><th>Active</th><th></th></tr></thead>
  <tbody>
  @foreach (var it in Model.Result.Items)
  {
    <tr>
      <td>@it.Id</td>
      <td>@it.Name</td>
      <td>@it.IsActive</td>
      <td>
        <a asp-page="./Edit" asp-route-id="@it.Id">Edit</a> |
        <a asp-page="./Delete" asp-route-id="@it.Id">Delete</a>
      </td>
    </tr>
  }
  </tbody>
</table>

<div>
  Page @Model.Result.Page of @Math.Ceiling((double)Model.Result.Total/Model.Result.PageSize)
  @if (Model.Result.Page > 1) { <a asp-page="./Index" asp-route-page="@(Model.Result.Page-1)" asp-route-pageSize="@Model.Result.PageSize" asp-route-q="@Model.Q" asp-route-sortBy="@Model.SortBy" asp-route-desc="@Model.Desc">Prev</a> }
  @if (Model.Result.Page * Model.Result.PageSize < Model.Result.Total) { <a asp-page="./Index" asp-route-page="@(Model.Result.Page+1)" asp-route-pageSize="@Model.Result.PageSize" asp-route-q="@Model.Q" asp-route-sortBy="@Model.SortBy" asp-route-desc="@Model.Desc">Next</a> }
</div>
```

``` cs title=""
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using MyApp.Contracts;

public class EditModel : PageModel
{
    private readonly ICrudApi<ItemDto, int> _items;
    public EditModel(IApiClientFactory factory) => _items = factory.Create<ItemDto, int>("items");

    [BindProperty] public ItemDto Form { get; set; } = new();
    [TempData] public string? Flash { get; set; }

    public async Task<IActionResult> OnGetAsync(int id)
    {
        var resp = await _items.GetAsync(id);
        if (resp.Status != ResponseStatusEnum.Success || resp.Data is null)
        {
            Flash = resp.Message ?? "Not found";
            return RedirectToPage("./Index");
        }
        Form = resp.Data;
        return Page();
    }

    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid) return Page();
        var resp = await _items.UpdateAsync(Form.Id, Form);
        if (resp.Status == ResponseStatusEnum.Success)
        {
            Flash = resp.Message ?? "Updated";
            return RedirectToPage("./Index");
        }
        ModelState.AddModelError(string.Empty, resp.Message ?? "Update failed");
        return Page();
    }
}
```
