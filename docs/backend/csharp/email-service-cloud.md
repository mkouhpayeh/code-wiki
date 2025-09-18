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
  Task<Response> SendMultipleEmails();
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
    var client = new SendGridClient(apiKey);
    var from = new EmailAddress(senderEmailAddress, senderName);
    var subject = payload.Subject;
    var to = new EmailAddress(receiverEmailAddress, $"{payload.Firstname} {payload.Lastname}");
    var textContent = payload.Body;
    var htmlContent = $"<strong>{payload.Body}</strong>";
    var msg = MailHelper.CreateSingleEmail(from, to, textContent, htmlContent);
    var response = client.SendEmailAsync(msg);
    response.wait();
    var result = response.Result;
    return response;
  }

  public Task<Response> SendMultipleEmails()
  {
    var apiKey = _configuration.GetSection("SendGrid")["ApiKey"];
    var client = new SendGridClient(apiKey);
    var from = new EmailAddress(senderEmailAddress, senderName);
    var subject = "Test Sub";
    var tos = new List<EmailAddress>(){
      new EmailAddress("Email1", "Name1"),
      new EmailAddress("Email2", "Name2")
    };
    var textContent = "Body";
    var htmlContent = $"<strong>Body</strong>";
    var msg = MailHelper.CreateSingleEmailToMultipleRecipients(from, tos, textContent, htmlContent);
    var response = client.SendEmailAsync(msg);
    response.wait();
    var result = response.Result;
    return response;
  }
}
```
```cs title="program.cs"
builder.Services.AddTransient<IEmailService, EmailService>();
```
