# Azure Application Insights

## Install Packages 
```  cs
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

## Configure
``` cs Title="Program.cs"
using Microsoft.ApplicationInsights;

var builder = WebApplication.CreateBuilder(args);

// Enable Application Insights
builder.Services.AddApplicationInsightsTelemetry(
    builder.Configuration["ApplicationInsights:ConnectionString"]);

builder.Services.AddSingleton<WeatherService>();

var app = builder.Build();

app.MapGet("/", () => "Hello World with App Insights!");

app.MapGet("/weather", (WeatherService service) =>
{
    service.GetWeather();
    return Results.Ok("Weather checked and telemetry sent!");
});

app.Run();

public class WeatherService
{
    private readonly TelemetryClient _telemetry;

    public WeatherService(TelemetryClient telemetry)
    {
        _telemetry = telemetry;
    }

    public void GetWeather()
    {
        // Track custom event
        _telemetry.TrackEvent("GetWeatherCalled");

        // Track custom metric
        _telemetry.GetMetric("WeatherRequests").TrackValue(1);

        // Simulate error
        try
        {
            throw new Exception("Test Exception from WeatherService!");
        }
        catch (Exception ex)
        {
            _telemetry.TrackException(ex);
        }
    }
}

```

## Add ConnectionString
``` cs Title="appsettings.json"
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx;IngestionEndpoint=https://<region>.in.applicationinsights.azure.com/"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

- Visit /weather → This will trigger events, metrics, and a sample exception.

- Open Azure Portal → Application Insights → Logs / Live Metrics.

- You’ll see your requests, events, and errors streaming in real-time. 🎉

## Querying Telemetry with KQL
-Application Insights uses Kusto Query Language (KQL) for analyzing telemetry data.<br> For example, 

### see all failed requests:
```
requests
| where success == false
| order by timestamp desc
```
### find the top 5 slowest endpoints
```
requests
| summarize avg(duration) by name
| top 5 by avg_duration desc
```

