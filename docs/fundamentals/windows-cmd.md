# Windows Command

## Copy files
```cmd
robocopy "C:\SourceFolder" "Z:\DestinationFolder" /E /Z /MT:32 /R:1 /W:1
```

**Options**  
`/E` → Copy subfolders, including empty ones.  
`/Z` → Restartable mode (resumes if interrupted).  
`/MT:32` → Multi-threaded copy (32 threads; up to 128).  
`/R:1` → Retry 1 time on copy failure.  
`/W:1` → Wait 1 second between retries.  
`/LOG:file.txt` → Write operation log to `file.txt`.  
`/MIR` → Mirror source to destination (WARNING: can delete files at destination).

**Notes**
- Use an elevated command prompt (Run as Administrator) if you encounter permission errors.
- For large transfers consider increasing `/MT` but monitor CPU and disk I/O.
- When using network drives, map them first or use UNC paths (`\\server\share`).

## Get PID / Find running process
```cmd
tasklist /fi "IMAGENAME eq AsyncTest.exe"
```

**Tip**  
To get more details including the PID, use:
```cmd
tasklist /v /fi "IMAGENAME eq AsyncTest.exe"
```
Or filter by window title or username:
```cmd
tasklist /fi "WINDOWTITLE eq MyApp*" /fi "USERNAME eq DOMAIN\User"
```

## TLS protocol

### Show/Set TLS

``` shell
# Show current value
[Net.ServicePointManager]::SecurityProtocol

# Force .NET to use TLS 1.2 only
[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
```

### Check Enabled Protocol
[SSL Test](https://www.ssllabs.com/ssltest/analyze.html)  

[nartac](https://www.nartac.com/Products/IISCrypto/Download)

### Check Cipher Suites
``` shell
Get-TlsCipherSuite
```

## Set Env Variable
If you had Visual Studio running while you were setting the environment variables, you have to restart Visual Studio so that the new environment variables can take effect.

### CMD
``` shell
setx AZURE_OPENAI_API_KEY "xxxxxxxxxxxx"
```

### PowerShell
``` shell title="current user"
[Environment]::SetEnvironmentVariable("AZURE_OPENAI_API_KEY", "xxxxxxxxxxxx", "User")
```

``` shell title="system-wide (needs admin rights)"
[Environment]::SetEnvironmentVariable("AZURE_OPENAI_API_KEY", "xxxxxxxxxxxx", "Machine")
```

``` shell title="Temporary (current session)"
$env:AZURE_OPENAI_API_KEY = "xxxxxxxxxxxx"
```
