# Gitlab CI/CD

---

## 1. Deploy in IIS
Run test if available on main branche and deploy project manually on IIS when any changes applied to the production branch
- Create GitLab Repository
- Create VS Project and add the project to the source control
- Create CI file in the project root
``` yaml title =".gitlab-ci.yml"
stages: [build, test, package]

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
    - dotnet publish Hello-World.csproj -c $CONFIGURATION -o publish
  needs: [build]
  artifacts:
    paths:
      - publish/**
    expire_in: 7 days
  rules:
    - if: '$CI_COMMIT_BRANCH == "production"'
  when: on_success
```
- 
