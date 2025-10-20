# CI/CD 

CI/CD is a set of practices and automation that let teams integrate code frequently, build and test artifacts automatically, and deploy them to environments reliably.

---

## 1. Overview

- **Continuous Integration (CI):** Developers frequently merge code into a shared repository. Each merge (or push) triggers automated checks such as linting, static analysis, and unit tests so problems are discovered early.
- **Continuous Delivery (CD - delivery):** After successful CI, code is packaged into a build artifact (a package or container image). Additional higher-level tests (integration, acceptance) run and the artifact is stored in a registry so it’s always ready to be released.
- **Continuous Deployment (CD - deployment):** Builds that pass all automated checks are deployed automatically to production without human intervention. This requires strong test coverage and a robust pipeline of validations.

---

### CI/CD Concepts

- **Workflow** — YAML file that defines automation (triggers, jobs).
- **Event / Trigger** — What starts a workflow (push, pull_request, schedule, workflow_call).
- **Runner** — The compute environment where jobs run (Ubuntu, Windows, macOS).
- **Job** — A group of steps run on a runner.
- **Step** — A single command, script, or action executed inside a job.
- **Action** — A reusable unit (Docker container or JavaScript) to perform a task.
- **Matrix strategy** — Runs the same job multiple times with different parameter values (e.g., different OS or language versions).

---

## 2. Typical process

- Developer pushes code to repository → workflow triggered.

- CI workflow:
    - Lint / static analysis
    - Unit tests
    - Build verification

- Delivery workflow:
    - Build artifact (package/container image)
    - Run integration/acceptance tests
    - Publish to a registry (artifact repository)

- Deployment workflow:
    - Retrieve artifact from registry
    - Deploy to target environment (staging → production)
    - Optional rollout strategies (canary, blue/green)

**Notes:**
- Triggering can be per push, on tags, on releases, or on branch events depending on release policy.
- Permissions must be configured for workflows that read/write registries (e.g., GitHub Packages) — e.g., `packages: write`.

---

## 3. Artifacts & Registries

- **Artifacts** are the output of a build (packages, container images, etc.).

- **Package formats / registries by ecosystem:**
    - Java → JAR/Gradle/Maven (registry: Maven Central or private Nexus/Artifactory)
    - JavaScript → npm (registry: npm or GitHub Packages)
    - .NET → NuGet (registry: nuget.org or private feed)
    - Docker → container image (registry: Docker Hub, GitHub Container Registry, ECR)

- **Best practice:** Bump the package version on each release so each published artifact has a unique version.

**Common steps when publishing packages**

1. Authenticate with the registry (handled by build steps).
2. Build the package/artifact.
3. Publish the package to the target registry.
4. Ensure configuration files refer to the correct registry and version (e.g., `pom.xml`, `package.json`, `.csproj`, `Dockerfile`).

---

## 4. Reusable workflows

To make a workflow callable from another workflow:

```yaml
# .github/workflows/ci.yml
on:
  workflow_call:
    inputs:
      build-type:
        required: false
        type: string
```

Call it from another repo:

```yaml
jobs:
  integration:
    uses: usernme/repoName/.github/workflows/ci.yml@main
    with:
      build-type: "full"
    permissions:
      contents: read

  build:
    needs: [integration]
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      # ...
```

---

## 5. Deploying: Environments & protections

- Use **environments** to represent targets: `staging`, `production`, etc.
- Reference an environment in a job:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        run: ./deploy.sh
```

- **Protection rules**: environments can enforce branch filters and required reviewers before deployments proceed.
- **Variables & secrets**: can be scoped at repo or environment level. Use secrets for credentials:

```yaml
- name: Configure AWS Credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws-region: ${{ vars.AWS_REGION }}
```

- **Concurrency**: prevent conflicting deployments by grouping:

```yaml
concurrency:
  group: deploy-production
  cancel-in-progress: false
```

Or per-job concurrency:

```yaml
jobs:
  deploy_staging:
    environment: Staging
    concurrency:
      group: deploy-staging
      cancel-in-progress: true
```

---

## 6. Recommended practices

- Keep CI fast: split quick checks (lint, unit tests) from long-running tests (integration).
- Fail early: run lightweight validation first so more expensive steps only run on passing builds.
- Version artifacts reliably (semantic versioning where possible).
- Use immutable artifacts (tag images with commit SHA and version).
- Keep secrets out of logs and secure them in the platform’s secret store.
- Make workflows reusable and parameterizable with `workflow_call` and inputs.
- Monitor pipelines and add alerts on failed deployments or repeated flakiness.

---

## 7. GitHub Pages

- Build
    - **Jeykyll** is the default static site generator for GitHub Pages. The starter workflow is provided.
  
    - There are other popular static site generators but for these tools, we would create a custom workflow, with actions that run the selected site generator instead of Jekyll. 
        - Hugo
        - Gatsby 

- Deploy
    - The deployment job transfers the artifact to the GitHub Pages service, where it can be accessed using a dedicated URL on the github.io domain (http(s)://<username>.github.io/<repository>). Sites deployed to GitHub Pages can also be configured to use a custom domain. 
   
    - Configure a workflow for GitHub Pages. Settings > Unser Code and Automating > Pages > Change source to GitHub Actions > With .md file, choose the Github Pages Jekyll

## 8. Service accounts

- Service accounts provide a secure way for our deployment workflows to interact with services outside of GitHub.

- Used to manage credentials and permissions

- Not associated with specific users

- Permissions are limited to specific tasks

- Credentials:
    - Username and password
    - API keu
    - Certificate
        
- Store credentials in environment secrets

- Configure workflows to access and use secrets
