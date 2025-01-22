# Google reCAPTCHA

## reCAPTCHA V2 component

### Get Google Keys
```
[Create here](https://www.google.com/recaptcha/admin/create)
```

### Install
``` sh
dotnet add package reCAPTCHA.AspNetCore --version 3.0.10
Install-Package reCAPTCHA.AspNetCore 
```

### DI
``` cs title="program.cs"
builder.Services.AddRecaptcha(options =>
{
  options.SiteKey = configuration.GetSection("reCAPTCHA:SiteKey").Value;
  options.SecretKey = configuration.GetSection("reCAPTCHA:SecretKey").Value;
});
```

### Add reference 
``` html
<script src="https://www.google.com/recaptcha/api.js?render=explicit" async defer></script>
<script src="/js/site.js"></script>
```

!!! note "Two Captcha in one page"

    set render=explicit
    No need to use two different siteKey and secret 
    set &hl=de to change language. 
  

### Create Page 
``` cs title="RecaptchaComponent.razor"
@using reCAPTCHA.AspNetCore
@inject IRecaptchaService RecaptchaService
@inject IJSRuntime JSRuntime

<div id="@RecaptchaDivId" class="g-recaptcha" data-sitekey="@SiteKey"></div>

@code {
	[Parameter]
	public string SiteKey { get; set; }
	
	private string RecaptchaToken;
	private string RecaptchaDivId;
	private int widgetId;
	
	protected override void OnInitialized()
	{
	    RecaptchaDivId = $"recaptcha_{Guid.NewGuid()}";
	}
	
	protected override async Task OnAfterRenderAsync(bool firstRender)
	{
	    if (firstRender)
	    {
	        widgetId = await JSRuntime.InvokeAsync<int>("renderRecaptcha", RecaptchaDivId, SiteKey);
	    }
	}
	
	public async Task<bool> ValidateAsync()
	{
	    RecaptchaToken = await JSRuntime.InvokeAsync<string>("getRecaptchaToken", widgetId);
	    var result = await RecaptchaService.Validate(RecaptchaToken);
	    return result.success;
	}
	
	 public async Task ResetRecaptchaAsync()
 {
     await JSRuntime.InvokeVoidAsync("grecaptcha.reset", widgetId);
 }
}
```

### Add .js file
```js
window.renderRecaptcha = (elementId, siteKey) => {
  var widgetId = grecaptcha.render(elementId, {
  'sitekey': siteKey
  });
  return widgetId;
  };
  
  window.getRecaptchaToken = async (widgetId) => {
  var recaptchaResponse = grecaptcha.getResponse(widgetId);
  return recaptchaResponse;
};
```

### Usage in page
``` csharp
@using reCAPTCHA.AspNetCore
@inject IRecaptchaService RecaptchaService

<RecaptchaComponent @ref="ForgotPasswordRecaptcha" SiteKey="***" />

@code{
	 @if (!String.IsNullOrEmpty(siteKey))
   {
     <RecaptchaComponent @ref="recaptchaComponent" SiteKey=@siteKey />
   }
}
```

### Validate
``` cs
if (await ForgotPasswordRecaptcha.ValidateAsync())
{
}
```
