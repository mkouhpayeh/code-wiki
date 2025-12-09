# CI/CD GitHub

CI/CD is a set of practices and automation that let teams integrate code frequently, build and test artifacts automatically, and deploy them to environments reliably.

---

## Overview

- **Continuous Integration (CI):** Developers frequently merge code into a shared repository. Each merge (or push) triggers automated checks such as linting, static analysis, and unit tests so problems are discovered early.
- **Continuous Delivery (CD - delivery):** After successful CI, code is packaged into a build artifact (a package or container image). Additional higher-level tests (integration, acceptance) run and the artifact is stored in a registry so it’s always ready to be released.
- **Continuous Deployment (CD - deployment):** Builds that pass all automated checks are deployed automatically to production without human intervention. This requires strong test coverage and a robust pipeline of validations.


**CI/CD Concepts**

- **Workflow** — YAML file that defines automation (triggers, jobs).
- **Event / Trigger** — What starts a workflow (push, pull_request, schedule, workflow_call).
- **Runner** — The compute environment where jobs run (Ubuntu, Windows, macOS).
- **Job** — A group of steps run on a runner.
- **Step** — A single command, script, or action executed inside a job.
- **Action** — A reusable unit (Docker container or JavaScript) to perform a task.
- **Matrix strategy** — Runs the same job multiple times with different parameter values (e.g., different OS or language versions).

---

## Typical process

- Developer pushes code to repository → workflow triggered.

- **CI workflow:**
    - Lint / static analysis
    - Unit tests
    - Build verification

- **Delivery workflow:**
    - Build artifact (package/container image)
    - Run integration/acceptance tests
    - Publish to a registry (artifact repository)

- **Deployment workflow:**
    - Retrieve artifact from registry
    - Deploy to target environment (staging → production)
    - Optional rollout strategies (canary, blue/green)

**Notes:**
- Triggering can be per push, on tags, on releases, or on branch events depending on release policy.
- Permissions must be configured for workflows that read/write registries (e.g., GitHub Packages) — e.g., `packages: write`.

---

## Artifacts & Registries

- **Artifacts** are the output of a build (packages, container images, etc.).

- **Package formats / registries by ecosystem:**
    - Java → JAR/Gradle/Maven (registry: Maven Central or private Nexus/Artifactory)
    - JavaScript → npm (registry: npm or GitHub Packages)
    - .NET → NuGet (registry: nuget.org or private feed)
    - Docker → container image (registry: Docker Hub, GitHub Container Registry, ECR)

- **Best practice:** Bump the package version on each release so each published artifact has a unique version.

- **Common steps when publishing packages**
    1. Authenticate with the registry (handled by build steps).
    2. Build the package/artifact.
    3. Publish the package to the target registry.
    4. Ensure configuration files refer to the correct registry and version (e.g., `pom.xml`, `package.json`, `.csproj`, `Dockerfile`).

---

## Reusable workflows

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

## Deploying: Environments & protections

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

## Recommended practices

- Keep CI fast: split quick checks (lint, unit tests) from long-running tests (integration).
- Fail early: run lightweight validation first so more expensive steps only run on passing builds.
- Version artifacts reliably (semantic versioning where possible).
- Use immutable artifacts (tag images with commit SHA and version).
- Keep secrets out of logs and secure them in the platform’s secret store.
- Make workflows reusable and parameterizable with `workflow_call` and inputs.
- Monitor pipelines and add alerts on failed deployments or repeated flakiness.

---

## GitHub Pages

- Build
    - **Jeykyll** is the default static site generator for GitHub Pages. The starter workflow is provided.
  
    - There are other popular static site generators but for these tools, we would create a custom workflow, with actions that run the selected site generator instead of Jekyll. 
        - Hugo
        - Gatsby 

- Deploy
    - The deployment job transfers the artifact to the GitHub Pages service, where it can be accessed using a dedicated URL on the github.io domain (http(s)://<username>.github.io/<repository>). Sites deployed to GitHub Pages can also be configured to use a custom domain. 
   
    - Configure a workflow for GitHub Pages. Settings > Unser Code and Automating > Pages > Change source to GitHub Actions > With .md file, choose the Github Pages Jekyll
 

Two common approaches to publish a static site with GitHub Pages using GitHub Actions:

A. **Official Pages workflow** — upload artifact + deploy (recommended when you want to use GitHub Pages as the publishing platform and leverage the Pages API). This is the modern, supported flow using `actions/upload-pages-artifact` and `actions/deploy-pages`.

**How it works (overview)**
1. Build your static site into a directory (for example `./public` or `./site`).
2. Upload the build output as a Pages artifact using `actions/upload-pages-artifact@v1`.
3. Deploy the uploaded artifact to GitHub Pages using `actions/deploy-pages@v1`.

**Example: generic static-site workflow**
```yaml
name: Publish site
on:
  push:
    branches: [ main ]

permissions:
  contents: read
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js (if your site uses Node)
        uses: actions/setup-node@v4
        with:
          node-version: '18'

      - name: Install dependencies and build
        run: |
          npm ci
          npm run build   # produces ./public or ./dist

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v1
        with:
          path: ./public
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        uses: actions/deploy-pages@v1
```

**Notes & tips for the official flow**
- Make sure `path` points to the directory containing the static files (html, css, js). The upload step packages them for deployment. If your SSG produces a different folder (e.g., `public`, `dist`, `site`), set `path` accordingly.
- The artifact must be a gzip-compressed tar under 10 GB and must not contain symlinks. The upload action handles packaging automatically.
- The `pages: write` permission is required for the deploy job.
- You can keep build and deploy in the same job but splitting them (build job + deploy job) provides clearer logs and separation of concerns.
- For Jekyll specifically (see below) you may prefer to let GitHub handle Jekyll builds — but if you need custom plugins or Ruby gems, build in Actions and use the artifact flow to publish the resulting `_site` directory.

B. **Deploy-to-branch flow (peaceiris / gh-pages branch)** — push built static files to the `gh-pages` branch (commonly used before the official actions existed and still popular for simple setups). Use `peaceiris/actions-gh-pages` or a deploy key for authenticated push. This method creates/updates the publishing branch (e.g., `gh-pages`) and GitHub Pages serves the content from that branch.

**Example: push to `gh-pages` using peaceiris action**
```yaml
name: Deploy to gh-pages
on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build static site
        run: |
          npm ci
          npm run build   # produces ./public
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

**Notes for deploy-to-branch**
- `peaceiris/actions-gh-pages` supports using a deploy key (`deploy_key`) if you prefer not to use the repo `GITHUB_TOKEN` for pushes.
- This flow will create commits on the `gh-pages` branch (or another branch you configure) and GitHub Pages can be set to use that branch as the publishing source.

- Jekyll (GitHub's built-in) vs building in Actions

    - GitHub Pages automatically builds Jekyll sites if you enable Pages and have an appropriate `_config.yml`. That is convenient but limited: GitHub Pages only allows a vetted set of plugins and an older Ruby/Jekyll toolchain.
    - If you need custom Jekyll plugins or a newer toolchain, build the site in Actions (e.g., `bundle install` + `bundle exec jekyll build`) and use the artifact/deploy flow to publish the resulting `_site` directory.

- Custom domain & HTTPS
    - Add a `CNAME` file to the root of the published site (or configure the custom domain in repository → Settings → Pages). The `CNAME` file must contain only the domain name (e.g., `docs.example.com`).
    - Configure DNS: create an `A` record pointing to GitHub Pages IP addresses (if apex/root domain) and/or a `CNAME` pointing to `<username>.github.io` for subdomains. GitHub docs list the exact IPs and recommended records.
    - After adding a custom domain, enable *Enforce HTTPS* in Pages settings once the DNS and TLS provisioning completes (ACME provisioning is automatic for supported domains).
    - If you use a custom certificate or external CDN, follow GitHub Pages docs for advanced configurations.

- Useful tips & gotchas
    - If your site contains files or folders that start with an underscore (e.g., `_assets`) and you **do not** use Jekyll, include a `.nojekyll` file in the published root to prevent GitHub from running Jekyll processing.
    - For MkDocs, Hugo, or other SSGs, build in Actions (recommended) and publish the output directory.
    - For private repositories, GitHub Pages can publish from Actions; ensure the workflow has correct `pages: write` permission. Using `GITHUB_TOKEN` or deploy keys works — check the action docs for authentication options.
    - To speed up deployments, avoid committing build artifacts to the default branch; build in Actions and deploy artifacts to Pages or `gh-pages` branch.
    - Monitor the Actions run logs and the Pages deploy history (Settings → Pages → Deployment history) to debug failures.

---

## Service accounts

- Service accounts provide a secure way for our deployment workflows to interact with services outside of GitHub.

- Used to manage credentials and permissions

- Not associated with specific users

- Permissions are limited to specific tasks

- Credential types:
    - Username and password
    - API key
    - Certificate
        
- Store credentials in environment secrets

- Configure workflows to access and use secrets

## Terraform
Terraform is a widely adopted open-source tool for working with infrastructure as code. When we run Terraform, a report is generated called the **Terraform plan**. The plan describes any changes that need to be made to produce the desired state according to the configuration in the repo. We can use command-line tools, like **awk** and **sed**, to modify the Terraform plan into a report using Markdown styling. Then we can write the report to **GITHUB_STEP_SUMMARY** so the plan can be quickly and easily viewed before it gets applied. 
- We need to update the permissions on the service account. We should add the **AmazonS3FullAccess permission**. Terraform will use the service account credentials to read and write state files to an S3 bucket. We need also to create a bucket to hold the Terraform state files. 
