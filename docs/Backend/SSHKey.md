## Create SSH Key command

- Here is the command to create SSH key. If you want to start to push your projects to GIT, it's the first step.  
- when you logged in to GIT website you will see a notification to add your SSH key! run this command with PowerShell or CMD...  
============================================================
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f "C:\path\to\your\folder\your_key_name"
```
============================================================  
- replace the email and the path and then edit the generated .pub file in the specified path, with notepad and copy all text to paste in Gitlab website.  
