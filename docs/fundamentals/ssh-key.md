# 📚 SSH Key

Use this guide to create an SSH key on Windows and add it to your Git hosting service for secure, password-less pushes.

---

## 1️⃣ Create SSH Key

- Here is the command to create an SSH key. If you want to start pushing your projects to GIT, it's the first step.  
- When you log in to the GIT website you will see a notification to add your SSH key. Run this command in PowerShell or CMD:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f "C:\path\to\your\folder\your_key_name"
```

- Replace the email and the path. Then open the generated `.pub` file at the specified path with Notepad, copy all text, and paste it into the GitLab/GitHub website.

**Optional (recommended):** use `ed25519` for smaller keys and better security:
```bash
ssh-keygen -t ed25519 -C "your_email@example.com" -f "C:\path\to\your\folder\your_ed25519_key"
```

---

## 2️⃣ Check for existing Key

```bash
dir C:\Users\YourUsername\.ssh
```

---

## 3️⃣ Copy public key (Windows)

```powershell
type C:\path\to\your\folder\your_key_name.pub | clip
```

---

## 4️⃣ Notes

- Keep your **private** key (`your_key_name` without `.pub`) secure and never share it.
- Use `ssh-agent` to load keys for the session:
```powershell
# Start the agent (PowerShell)
Start-Service ssh-agent
ssh-add C:\path\to\your\folder\your_key_name
```

---
