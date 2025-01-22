# Email Service

## .Net Core

``` shell
Install-Package MimeKit
Install-Package MailKit
```

``` csharp
 public interface IEmailSender
 {
     Task SendEmailAsync(string emailAddress, string subject, string body);
 }
```

``` csharp
  public class MyEmailSender : IEmailSender
  {
     private readonly IConfiguration _config;

     public MyEmailSender(IConfiguration config)
     {
         _config = config;
     }

     public MyEmailSender () : this(new ConfigurationBuilder().Build())
     {
         _config = new ConfigurationBuilder().Build();
     }

     public async Task SendEmailAsync(string emailAddress, string subject, string body)
     {
        try
        {
            var message = new MimeMessage();
    
            // Configure the sender
            message.From.Add(new MailboxAddress(".::GrammFit::.", 
                            _config.GetSection("EmailSettings:NoReply:SmtpAddress").Value));
            // Configure the recipient
            message.To.Add(new MailboxAddress(string.Empty, emailAddress));
            // Set the subject
            message.Subject = subject;
    
            // Add the email body
            var textPart = new TextPart(MimeKit.Text.TextFormat.Html)
            {
                Text = body
            };
    
            message.Body = textPart;
    
            using (var client = new SmtpClient())
            {
                // Set up server certificate validation callback
                client.ServerCertificateValidationCallback = (s, c, h, e) => true;
    
                // Connect to the SMTP server with the appropriate security options
                await client.ConnectAsync(
                    _config.GetSection("EmailSettings:NoReply:SmtpServer").Value,
                    int.Parse(_config.GetSection("EmailSettings:NoReply:SmtpPort").Value ?? "587"),
                    SecureSocketOptions.StartTls);
    
                // Authenticate with the server
                await client.AuthenticateAsync(
                    _config.GetSection("EmailSettings:NoReply:SmtpUsername").Value,
                    _config.GetSection("EmailSettings:NoReply:SmtpPassword").Value);
    
                // Send the email
                await client.SendAsync(message);
    
                // Disconnect cleanly
                await client.DisconnectAsync(true);
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Email sending error: {ex}");
            throw;
        }
     }
   }
```

``` csharp title="program.cs"
 builder.Services.AddTransient<MyEmailSender>();
```

## ASP.Net MVC
``` csharp
using System.Net.Mail;
using System.Net.Mime;
 
 SmtpClient mailer = new SmtpClient();
 
 public bool SendMail(string mailTo, string subject, string body, Stream attachment= null)
 {
     if (!ValidateMailAddress(mailTo)) { return false; }


     try
     {

         ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls13 | SecurityProtocolType.Tls12;
         var login = new NetworkCredential();
         login.UserName = "noreply";
         login.Password = "***";
         mailer.UseDefaultCredentials = false;
         mailer.Host = "mail.****.com";
         mailer.Port = 587;
         mailer.EnableSsl = false;
         mailer.Credentials = login;

         var mail = new MailMessage();
         mail.From = new MailAddress("noreply@***.com");
         mail.To.Add(mailTo);
         mail.Subject = subject;
         mail.Body = body;
         mail.IsBodyHtml = true;

         if (attachment!= null)
         {
             var data = new Attachment(attachment, MediaTypeNames.Application.Octet);
             var disposition = data.ContentDisposition;
             disposition.CreationDate = DateTime.Now;
             disposition.ModificationDate = DateTime.Now;
             disposition.ReadDate = DateTime.Now;
             mail.Attachments.Add(data);
         }

         mailer.Send(mail);
         return true;
     }
     catch (Exception)
     {
         return false;
     }
 }
```
