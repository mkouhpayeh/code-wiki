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

---

## 1. Deploy .Net Web in IIS
Run test if available on main branche and deploy project manually on IIS when any changes applied to the production branch
- Create GitLab Repository
  
- Create VS Project and add the project to the source control

- Create CI file in the project root

```yaml title =".gitlab-ci.yml"
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

-  
