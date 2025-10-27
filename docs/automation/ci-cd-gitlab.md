# CI/CD GitLab

- CI/CD pipelines are the fundamental component of GitLab CI/CD. Pipelines are configured in a ".gitlab-ci.yml" file by using YAML keywords.
  
- Pipelines can run automatically for specific events, like when pushing to a branch, creating a merge request, or on a schedule. When needed, you can also run pipelines manually.

- You can add CI/CD variables to a project’s settings. Projects can have a maximum of 8000 CI/CD variables.
    - **Key**: Must be one line, with no spaces, using only letters, numbers, or _.
    - **Value**: The value is limited to 10,000 characters, but also bounded by any limits in the runner’s operating system. The value has extra limitations if Visibility is set to Masked or Masked and hidden.
    - **Type**: Variable (default) or File.
    - **Environment scope**: Optional. All (default) (*), a specific environment, or a wildcard environment scope.
    - **Protect variable**: Optional. If selected, the variable is only available in pipelines that run on protected branches or tags.
    - **Visibility**: Select Visible (default), Masked, or Masked and hidden.
    - **Expand variable reference**: Optional. If selected, the variable can reference another variable. It is not possible to reference another variable if Visibility is set to Masked or Masked and hidden.

- Runners are the agents that run the GitLab Runner application, to execute GitLab CI/CD jobs in a pipeline. They are responsible for running your builds, tests, deployments, and other CI/CD tasks defined in .gitlab-ci.yml files.

---

## Deploy .Net Web in IIS
Run test if available on main branche and deploy project manually on IIS when any changes applied to the production branch.

1. One-time IIS server prep. Do these on the remote server:
    - Install [.NET Hosting Bundle](https://dotnet.microsoft.com/en-us/download/dotnet/9.0) (match your target, e.g. .NET 9). Reboot if the installer asks.
   
    - Create the site folder
   
    ```
    New-Item -ItemType Directory -Force -Path "C:\inetpub\wwwroot\Hello-World"
    ```
    
    - Create a deploy user (least-privilege)
   
    ```
    New-LocalUser -Name "gitlab_deploy" -Password (Read-Host -AsSecureString "Password") -FullName "GitLab Deploy User" -PasswordNeverExpires
    Add-LocalGroupMember -Group "Remote Management Users" -Member "gitlab_deploy"
    # File write permission on the site folder:
    $acl = Get-Acl "C:\inetpub\wwwroot\Hello-World"
    $rule = New-Object System.Security.AccessControl.FileSystemAccessRule("gitlab_deploy","Modify","ContainerInherit, ObjectInherit","None","Allow")
    $acl.AddAccessRule($rule)
    Set-Acl "C:\inetpub\wwwroot\Hello-World" $acl
    ```

    ```
    Add-LocalGroupMember -Group "IIS_IUSRS" -Member "gitlab_deploy"
    ```
    
    - Enable WinRM + HTTPS listener (port 5986)
  
    ```
    # Enable WinRM
    Enable-PSRemoting -Force
    
    # Create a self-signed cert for WinRM HTTPS
    $cert = New-SelfSignedCertificate -DnsName "ServerName" -CertStoreLocation "Cert:\LocalMachine\My"

    # Create HTTPS listener using that cert
    $thumb = $cert.Thumbprint
    winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"ServerName`"; CertificateThumbprint=`"$thumb`"}"

    # Open firewall
    New-NetFirewallRule -DisplayName "WinRM HTTPS (5986)" -Direction Inbound -Protocol TCP -LocalPort 5986 -Action Allow
    ```

    - Verify:
  
    ```
    winrm enumerate winrm/config/Listener
    Test-NetConnection -ComputerName ServerName -Port 5986
    ```


2. On the Windows GitLab Runner machine (not the IIS server)
We need a Windows runner somewhere that can reach ServerName:5986:
    - Install & register GitLab Runner (executor = shell), tag it windows.
   
    - Allow connecting to self-signed/hostname-mismatch (we’ll pass flags in the script). (No global TrustedHosts needed because we use HTTPS + skip checks.)
   
    ```
    $sec = ConvertTo-SecureString "YourStrongPassword!" -AsPlainText -Force
    $cred = New-Object pscredential ("ServerName\gitlab_deploy", $sec)
    $so = New-PSSessionOption -SkipCACheck -SkipCNCheck
    New-PSSession -ComputerName ServerName -UseSSL -Credential $cred -SessionOption $so
    # If you get a session object (not an error), you’re good. Then:
    Get-PSSession | Remove-PSSession
    ```

3. Create CI file ".gitlab-ci.yml" in the project root

    ```
    stages: [build, test, package]

    # ---------- CI (build/test/publish) ----------
    image: mcr.microsoft.com/dotnet/sdk:9.0

    variables:
      DOTNET_CLI_TELEMETRY_OPTOUT: "1"
      DOTNET_SKIP_FIRST_TIME_EXPERIENCE: "1"
      CONFIGURATION: "Release"

    cache:
      key: "nuget"
      paths:
        - .nuget/packages

    build:
      stage: build
      script:
        - dotnet --info
        - dotnet restore
        - dotnet build -c $CONFIGURATION
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'
        - if: '$CI_COMMIT_BRANCH == "production"'

    test:
      stage: test
      script:
        - if [ -d "tests" ]; then dotnet test --no-build -c $CONFIGURATION; else echo "No tests found, skipping."; fi
      needs: [build]
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'

    package:
      stage: package
      script:
        # publish to a fixed folder that we artifact
        - dotnet publish Hello-World.csproj -c $CONFIGURATION -o publish
      needs: [build]
      artifacts:
        paths:
          - publish/**          
          - scripts/**
        expire_in: 7 days
      rules:
        - if: '$CI_COMMIT_BRANCH == "production"'
      when: on_success
    ```
    
4. Register Runner
    -  [Download Runner](https://docs.gitlab.com/runner/install/windows.html)
    -  Register Runner to the Gitlab project
        - Settings > CI/CD > Runner > Tag: windows
      
        ```bash
        .\gitlab-runner.exe register  --url https://gitlab-URL  --token xxxxxxxxxxxxxx
        # enter the URL
        # enter the name
        # enter executer: shell
        # edit the config.toml => shell = "powershell"
        ```
        
        ```bash
        .\gitlab-runner.exe run
        ```
    - The runner service will run as gitlab-runner user
   
        - Make sure that user has write permission to the IIS site folder (e.g. C:\inetpub\wwwroot\HelloWorld).

        - Make sure the IIS server already has the [.NET 9 Hosting Bundle](https://dotnet.microsoft.com/en-us/download/dotnet/9.0) installed.

5. Add deploy script to the repo
    Create folder scripts/ and add Deploy-IIS-RemoteWinRM.ps1:
    ```
    param(
      [Parameter(Mandatory=$true)][string]$PackageDir,          # local publish dir (from artifacts)
      [Parameter(Mandatory=$true)][string]$ComputerName,        # ServerName
      [Parameter(Mandatory=$true)][string]$SitePath,            # e.g. C:\inetpub\wwwroot\Hello-World
      [Parameter(Mandatory=$false)][string]$AppPool,            # optional, e.g. 'HelloWorldPool'
      [Parameter(Mandatory=$true)][string]$Username,            # e.g. ServerName\gitlab_deploy
      [Parameter(Mandatory=$true)][string]$Password
    )

    $ErrorActionPreference = 'Stop'

    # Prepare credentials & session
    $sec = ConvertTo-SecureString $Password -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential ($Username, $sec)
    $so = New-PSSessionOption -SkipCACheck -SkipCNCheck

    # Create a zip of the publish folder
    $zip = Join-Path $env:TEMP ("app_" + [guid]::NewGuid().Guid + ".zip")
    Add-Type -AssemblyName System.IO.Compression.FileSystem
    if (Test-Path $zip) { Remove-Item $zip -Force }
    [System.IO.Compression.ZipFile]::CreateFromDirectory($PackageDir, $zip)

    # Open remote session over WinRM HTTPS
    $session = New-PSSession -ComputerName $ComputerName -UseSSL -Credential $cred -SessionOption $so
    try {
      $remoteTemp = Invoke-Command -Session $session -ScriptBlock {
        $p = Join-Path $env:TEMP ("deploy_" + [guid]::NewGuid().Guid)
        New-Item -ItemType Directory -Path $p -Force | Out-Null
        $p
      }

      # Copy zip to remote
      Copy-Item -Path $zip -Destination (Join-Path $remoteTemp "app.zip") -ToSession $session

      # Remote unpack + mirror + recycle app pool
      Invoke-Command -Session $session -ScriptBlock {
        param($zipPath, $targetPath, $appPool)
        $ErrorActionPreference = 'Stop'

        if (!(Test-Path $targetPath)) { New-Item -ItemType Directory -Force -Path $targetPath | Out-Null }

        # Take app offline to avoid lock issues
        $offline = Join-Path $targetPath 'app_offline.htm'
        '<!DOCTYPE html><html><body><h3>Updating…</h3></body></html>' | Out-File -Encoding utf8 $offline

        if ($appPool) {
          Import-Module WebAdministration
          if (Test-Path "IIS:\AppPools\$appPool") { Stop-WebAppPool -Name $appPool }
        }

        Add-Type -AssemblyName System.IO.Compression.FileSystem
        $tmpExtract = Join-Path $env:TEMP ("extract_" + [guid]::NewGuid().Guid)
        New-Item -ItemType Directory -Force -Path $tmpExtract | Out-Null
        [System.IO.Compression.ZipFile]::ExtractToDirectory($zipPath, $tmpExtract)

        # Mirror files (fast & resilient)
        $rc = robocopy $tmpExtract $targetPath /MIR /NFL /NDL /NP /R:2 /W:2
        if ($LASTEXITCODE -ge 8) { throw "Robocopy failed with code $LASTEXITCODE" }

        Remove-Item $offline -ErrorAction SilentlyContinue

        if ($appPool) { Start-WebAppPool -Name $appPool }

        Remove-Item $tmpExtract -Recurse -Force
        Remove-Item $zipPath -Force
      } -ArgumentList (Join-Path $remoteTemp "app.zip"), $SitePath, $AppPool

    } finally {
      if ($session) { Remove-PSSession $session }
      if (Test-Path $zip) { Remove-Item $zip -Force }
    }

    Write-Host "Remote deploy complete."
    ```

7. Add protected CI/CD variables in GitLab
    - Project → Settings → CI/CD → Variables (mask & protect where appropriate):
      
        - PROD_SERVER = ws2019dggweb (or 192.168.35.15)
        
        - PROD_USER = ws2019dggweb\gitlab_deploy
     
        - PROD_PASSWORD = (the password you set)
     
        - IIS_SITE_PATH = C:\inetpub\wwwroot\Hello-World
     
        - (Optional) IIS_APPPOOL = your app pool name if you want to stop/start it
    
    - Mark them Protected so they’re only available to the production branch.

8. Extend the .gitlab-ci.yml with the deploy job (replace it):

    ```
    stages: [build, test, package, deploy]

    # ---------- CI (build/test/publish) ----------
    image: mcr.microsoft.com/dotnet/sdk:9.0

    variables:
      DOTNET_CLI_TELEMETRY_OPTOUT: "1"
      DOTNET_SKIP_FIRST_TIME_EXPERIENCE: "1"
      CONFIGURATION: "Release"

    cache:
      key: "nuget"
      paths:
        - .nuget/packages

    build:
      stage: build
      script:
        - dotnet --info
        - dotnet restore
        - dotnet build -c $CONFIGURATION
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'
        - if: '$CI_COMMIT_BRANCH == "production"'

    test:
      stage: test
      script:
        - if [ -d "tests" ]; then dotnet test --no-build -c $CONFIGURATION; else echo "No tests found, skipping."; fi
      needs: [build]
      rules:
        - if: '$CI_COMMIT_BRANCH == "main"'

    package:
      stage: package
      script:
        # publish to a fixed folder that we artifact
        - dotnet publish Hello-World.csproj -c $CONFIGURATION -o publish
      needs: [build]
      artifacts:
        paths:
          - publish/**           
          - scripts/**
        expire_in: 7 days
      rules:
        - if: '$CI_COMMIT_BRANCH == "production"'
      when: on_success

    # ---------- CD (manual deploy to IIS over WinRM) ----------
    # Requires a Windows GitLab Runner (executor: shell) tagged "windows"
    # and the repo file: scripts/Deploy-IIS-RemoteWinRM.ps1
    deploy_prod_remote_winrm:
      stage: deploy
      tags: ["windows"]                 # <-- Windows runner tag
      variables:
        GIT_STRATEGY: none             
      needs: [package]
      script:
        - >
          powershell -NoProfile -ExecutionPolicy Bypass -File
          scripts\Deploy-IIS-RemoteWinRM.ps1
          -PackageDir "$env:CI_PROJECT_DIR\publish"
          -ComputerName "$env:PROD_SERVER"
          -SitePath "$env:IIS_SITE_PATH"
          -AppPool "$env:IIS_APPPOOL"
          -Username "$env:PROD_USER"
          -Password "$env:PROD_PASSWORD"
      environment:
        name: production
        url: https://ServerName/      
      when: manual                      # ✅ requires confirmation click
      rules:
        - if: '$CI_COMMIT_BRANCH == "production"'

    ```

9. Protect the production path (optional but recommended)
    - Protect production branch: Settings → Repository → Protected Branches.
  
    - Environment approvals: Deployments → Environments → production → Protect, add at least one approver.

10. Test the full flow

    - Push to production branch (your pipeline will run build → package → deploy).
   
    - In Pipelines, click “Play” on deploy_prod_remote_winrm.
   
    - Watch the logs — it should:
   
        - Zip publish/
     
        - Open WinRM session to ws2019dggweb
     
        - Copy zip → extract → mirror into C:\inetpub\wwwroot\Hello-World
     
        - (Optionally) stop/start the app pool

    - Browse to http://ws2019dggweb/ (or the site binding you use) and verify the app.
