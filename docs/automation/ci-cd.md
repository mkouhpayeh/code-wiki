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


