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
    - Install .NET Hosting Bundle (match your target, e.g. .NET 9). Reboot if the installer asks.
   
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
    $cert = New-SelfSignedCertificate -DnsName "ws2019dggweb" -CertStoreLocation "Cert:\LocalMachine\My"

    # Create HTTPS listener using that cert
    $thumb = $cert.Thumbprint
    winrm create winrm/config/Listener?Address=*+Transport=HTTPS "@{Hostname=`"ws2019dggweb`"; CertificateThumbprint=`"$thumb`"}"

    # Open firewall
    New-NetFirewallRule -DisplayName "WinRM HTTPS (5986)" -Direction Inbound -Protocol TCP -LocalPort 5986 -Action Allow
    ```

    - Verify:
  
    ```
    winrm enumerate winrm/config/Listener
    Test-NetConnection -ComputerName ws2019dggweb -Port 5986
    ```


2. 
- Create GitLab Repository
  
- Create VS Project and add the project to the source control

- Create CI file ".gitlab-ci.yml" in the project root

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

- Add the deploy job to the .gitlab-ci.yml (replace it):

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
    GIT_STRATEGY: none              # only need artifacts from 'package'
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
    url: http://localhost/      # adjust if you have a hostname/binding
  when: manual                      # ✅ requires confirmation click
  rules:
    - if: '$CI_COMMIT_BRANCH == "production"'

```

```
```
