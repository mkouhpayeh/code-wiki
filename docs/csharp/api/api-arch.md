# API Architecture

---

## Several external APIs

- Named clients (string names like "BillingApi", "UserApi", …)
    - Good when:
        - You like the flexibility and don’t mind passing around IHttpClientFactory.
  
    - Register one AddHttpClient per API, each with its own base URL, headers, and policies:
      
    ``` cs title="Program.cs"
    var builder = WebApplication.CreateBuilder(args);

    // config in appsettings.json (recommended)
    var billingBase = builder.Configuration["Apis:Billing"];
    var userBase    = builder.Configuration["Apis:User"];
    var reportsBase = builder.Configuration["Apis:Reports"];
      
    static IAsyncPolicy<HttpResponseMessage> RetryPolicy() =>
        HttpPolicyExtensions
            .HandleTransientHttpError()
            .OrResult(r => r.StatusCode == HttpStatusCode.TooManyRequests)
            .WaitAndRetryAsync(3, retry => TimeSpan.FromMilliseconds(200 * (retry + 1)));
    
    builder.Services.AddHttpClient("BillingApi", client =>
    {
        client.BaseAddress = new Uri(billingBase);
        client.DefaultRequestHeaders.Accept.ParseAdd("application/json");
    })
    .AddPolicyHandler(RetryPolicy());
    
    builder.Services.AddHttpClient("UserApi", client =>
    {
        client.BaseAddress = new Uri(userBase);
        client.DefaultRequestHeaders.Accept.ParseAdd("application/json");
    })
    .AddPolicyHandler(RetryPolicy());
     
    builder.Services.AddHttpClient("ReportsApi", client =>
    {
        client.BaseAddress = new Uri(reportsBase);
        client.DefaultRequestHeaders.Accept.ParseAdd("application/json");
    })
    .AddPolicyHandler(RetryPolicy());

    ```

- Typed clients (one class per external API, injected directly)
    - Good when:
        - You want strong separation: one client per external system.
        - Easier to unit-test (you can mock BillingApiClient).
        - Cleaner DI in controllers/services (no need for IHttpClientFactory everywhere).
  
    - Define one class per external API and let DI inject a configured HttpClient into it.

    ```
    builder.Services.AddHttpClient<BillingApiClient>((sp, client) =>
    {
        var baseUrl = builder.Configuration["Apis:Billing"];
        client.BaseAddress = new Uri(baseUrl);
        client.DefaultRequestHeaders.Accept.ParseAdd("application/json");
    })
    .AddPolicyHandler(RetryPolicy());
    
    builder.Services.AddHttpClient<UserApiClient>((sp, client) =>
    {
        var baseUrl = builder.Configuration["Apis:User"];
        client.BaseAddress = new Uri(baseUrl);
        client.DefaultRequestHeaders.Accept.ParseAdd("application/json");
    })
    .AddPolicyHandler(RetryPolicy());
    
    // then your business services
    builder.Services.AddScoped<IBillingService, BillingService>();
    builder.Services.AddScoped<IUserService, UserService>();

    ```

    ``` cs title="Typed clients"
    public class BillingApiClient
    {
        private readonly HttpClient _http;
    
        public BillingApiClient(HttpClient http) => _http = http;
    
        public Task<HttpResponseMessage> GetInvoicesRawAsync(CancellationToken ct = default)
            => _http.GetAsync("api/invoices", ct);
    
        public async Task<T?> GetJsonAsync<T>(string url, CancellationToken ct = default)
        {
            var resp = await _http.GetAsync(url, ct);
            resp.EnsureSuccessStatusCode();
            return await resp.Content.ReadFromJsonAsync<T>(cancellationToken: ct);
        }
    }
    
    public class UserApiClient
    {
        private readonly HttpClient _http;
    
        public UserApiClient(HttpClient http) => _http = http;
    
        public Task<HttpResponseMessage> GetUserRawAsync(long id, CancellationToken ct = default)
            => _http.GetAsync($"api/users/{id}", ct);
    }
    ```

    ``` cs title="Business services using"
    public interface IBillingService
    {
        Task<IReadOnlyList<InvoiceDto>> GetInvoicesAsync(CancellationToken ct = default);
    }
    
    public class BillingService : IBillingService
    {
        private readonly BillingApiClient _api;
    
        public BillingService(BillingApiClient api) => _api = api;
    
        public async Task<IReadOnlyList<InvoiceDto>> GetInvoicesAsync(CancellationToken ct = default)
            => await _api.GetJsonAsync<IReadOnlyList<InvoiceDto>>("api/invoices", ct)
               ?? Array.Empty<InvoiceDto>();
    }

    ```

    ``` cs title="Controller"
    [ApiController]
    [Route("api/billing")]
    public class BillingController : ControllerBase
    {
        private readonly IBillingService _billing;
    
        public BillingController(IBillingService billing) => _billing = billing;
    
        [HttpGet("invoices")]
        public async Task<IActionResult> GetInvoices(CancellationToken ct)
        {
            var invoices = await _billing.GetInvoicesAsync(ct);
            return Ok(invoices);
        }
    }
    ```
