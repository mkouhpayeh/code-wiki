# Mailing Services
1- In-house email services
2- Cloud-based email services

## Cloud services benefits
- Lower cost
- Updates
- Security
- Pay as you go
- Expertise
- Scalability

## Common mailing services
- SendGrid
- Mandrill
- Mailgun

## Setup SendGrid
1- Create an account : https://sendgrid.com<br />
2- Create Sender Identity<br />
3- Reteieve an API Key<br />

### Development Env
1- Install Sendgrid NuGet package<br />
2- Add API Key to appsettings.json<br />
3- Add IEmailService.cs and EmailService.cs<br />

```cs title="appsettings.json"
"SendGrid":{
  "ApiKey" : "xxx"
},
```

```cs title="\Services\IEmailService.cs"
public class IEmailService
{
  Task<Response> SendSingleEmail(ComposeEmailVM payload);
}
```
```cs title="\Services\EmailService.cs"
public class EmailService: IEmailService
{
  private readonly IConfiguration _configuration;
  public EmailService(IConfiguration configuration)
  {
    _configuration = configuration;
  }
  public Task<Response> SendSingleEmail(ComposeEmailVM payload)
  {
    var apiKey = _configuration.GetSection("SendGrid")["ApiKey"];
    return null;
  }
}
```
```cs title="program.cs"
builder.Services.AddTransient<IEmailService, EmailService>();
```
