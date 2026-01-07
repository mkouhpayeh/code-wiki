# Mailing Services
1- In-house email services <br />
2- Cloud-based email services <br />

---

## Cloud services benefits
- Lower cost
- Updates
- Security
- Pay as you go
- Expertise
- Scalability

---

## Common mailing services
- SendGrid
- Mandrill
- Mailgun

---

## Setup SendGrid
1- Create an account : https://sendgrid.com <br />
2- Create Sender Identity <br />
3- Reteieve an API Key <br />

### Development Env
1- Install Sendgrid NuGet package <br />
2- Add API Key to appsettings.json <br />
3- Add IEmailService.cs and EmailService.cs <br />

```cs title="appsettings.json"
"SendGrid":{
  "ApiKey" : "xxx"
},
```

```cs title="\Data\ViewModels"
public class ComposeEmailModel
{
  public string Firstname {get;set;}
  public string Lastname {get;set;}
  public string Subject {get;set;}
  public string Email {get;set;}
  public string Body   {get;set;}

  public IFormFile Attachmetn {get;set;}
}
```

```cs title="\Services\IEmailService.cs"
public class IEmailService
{
  Task<Response> SendSingleEmail(ComposeEmailModel payload);
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
  public Task<Response> SendSingleEmail(ComposeEmailModel payload)
  {
    var apiKey = _configuration.GetSection("SendGrid")["ApiKey"];
    var client = new SendGridClient(apiKey);
    var from = new EmailAddress(senderEmailAddress, senderName);
    var subject = payload.Subject;
    var to = new EmailAddress(receiverEmailAddress, $"{payload.Firstname} {payload.Lastname}");
    var textContent = payload.Body;
    var htmlContent = $"<strong>{payload.Body}</strong>";
    var msg = MailHelper.CreateSingleEmail(from, to, textContent, htmlContent);
    if (payload.Attachment!=null && payload.Attachment.Length>0)
    {
      var fileContent ="";
      using(var reader = new StreamReader(payload.Attachment.OpenReadStream()))
      {
        fileContent = reader.ReadToEnd().ToString();
        byteData = Encoding.ASCII.GetBytes(fileContent);
      }
      Attachment attachment = new Attachment()
      {
        Content = Convert.ToBase64String(byteData);
        Filename = payload.Attachment.FileName,
        Type = payload.Attachment.ContentType,
        Disposition = "attachment"
      };
      msg.AddAttachment(attachment);
    }
    var response = client.SendEmailAsync(msg);
    response.wait();
    var result = response.Result;
    // Store the result in th DB
    IEnumerable<string> headerValues;
    var sendGridMessageID = string.Empty;
    if(result.Headers.TryGetValues("X-Message-ID", out headerValues))
    {
      sendGridMessageID = headerValues.FirstOrDefault();
      if (!string.IsNullOrEmpty(sendGridMessageID))
      {
        var newEmail = new Email()
        {
          EmailAddress = payload.Email,
          Fullname = $"{payload.Firstname} {payload.Lastname}",
          SendGridKey = sendGridMessageID,
          Status = "n/a"
        };
        _context.Email.Add(newEmail);
        _context.SaveChanges();
      }
    }
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

---

## Webhook
- Webhooks only work on HTTP or HTTPs and points which run on production envirinment.
- So to be able to run a localhost in a production-like environment, we use a toll named "ngrok"
- ngrok is a cross-platform application that enables developers tp expose a local development

---

## Setup ngrok 
1- Setup an Account: https://ngrok.com <br />
2- Install ngrok <br />
3- Create a Tunnel <br />
4- Run VS as Administrator > Tools > Start ngrok Tunnel <br />

---

## Add Event Webhook 
```cs title="SendGridEvents.cs"
public class SendGridEvents
{
  public string email { get; set; }
  public long timestamp { get; set; }
  public string @event { get; set; }
  [JsonPropertt("smtp-id")]
  public string smtp_id { get; set; }
  public string useragent { get; set; }
  public string ip { get; set; }
  public string sg_event_id { get; set; }
  public string sg_message_id { get; set; }
  public string reason { get; set; }
  public string status { get; set; }
  public string  response { get; set; }
  public string  tls { get; set; }
  public Uri url { get; set; }
  public string urloffset { get; set; }
  public string attempt { get; set; }
  public string category { get; set; }
  public string type { get; set; }
}
```

```cs title="Add an Empty API Controller"
private readonly AppDbContext _context;
public ctor (AppDbContext context)
{
  _context = context;
}
[HttpPost]
[Route("Webhook")]
public IActionResult WebHook (List<SendGridEvents> data)
{
  var debug = data;
  foreach(var item in data)
  {
    var dbEmail = _context.Email.FirstOrDefault(n => item.sg_message_id.StartsWith(n.SenGridKey));
    if(dbEmail!=null)
    {
      dbEmail.Status = item.@event;
      _context.SaveChanges();
    }
  }
  return Ok (debug);
}
```

