# CI/CD
Continuous Integration, Continuous Delivery, and Continuous deployment. </br>

- With continuous integration, developers work on their code in a local environment and commit their changes to a shared repository on a regular basis. Their code can then be combined, or in other words, integrated with code from other members of the team or any existing code. During the integration step, new code is tested and checked for any errors or any other requirements. This can include linting the code for syntax errors, and running unit tests on specific features. The goal of CI is to allow teams to identify, and resolve problems quickly, and early in the development process.
- Continuous Delivery, follows the integration step. After code has been integrated, the continuous delivery step automates the build process. This step can also include additional testing at a higher level. These tests might be more rigorous than unit tests and exercise multiple features at the same time. By the end of this step, the software will be packaged, and ready for release.  The goal of continuous delivery is to always have a version of the software that can be deployed into production as needed.
- Continuous deployment places software in a production environment without human interaction. In this case, the pipeline is fully automated with workflows using tests and validations to determine if the software is ready to be run in production. Using a well-defined continuous deployment pipeline, development teams can release software quickly and reliably.
- Process
  -   On every push to a code repository, continuous integration combines the new code with the existing code and runs tests. This can include linting, static analysis and unit tests.
  -   After integration is complete, continuous delivery builds the code into an artifact that can be used as part of a solution. This artifact is referred to as a package. Continuous delivery places the package in a location where it can be retrieved for use later. We'll also refer to the place where the package is stored as a registry.
  -   Once the package is stored in the registry, a deployable version of the software is available and ready to go.
  -   Each programming language has their very own package format and registry. (Java: JAR, JS=> npm, .Net=> NuGet, DockerFile=> Container image)
  -   Use this same approach for delivery workflows to create packages on every push. This approach provides a truly continuous way to deliver software as an artifact. But if we don't need delivery on every push, or if our software release process is more formal, we can use other triggers like creating a tag, creating a release, or pushing to a specific branch.
  -   In delivery workflows, we also need to set specific permissions to allow our actions to access the GitHub Packages service. This means we'll need to set permissions to write to the package registry. This permission needs to be added along with any other permissions needed, like reading the contents of the repo or creating checks in the actions interface.

- Keywords:
  -   Workflows define how all of the automation in a process is tied together including how the process gets started. Workflows are also coded using YAML syntax. 
  -   Events are the triggers that start a workflow. Triggers can be just about anything that happens while you're using GitHub, pushes to a repo, new pull requests, new issues, and so on.
  -   Runners define the compute layer where jobs are executed. GitHub Actions provides runners with popular operating systems like Ubuntu, Windows and Mac OS.
  -   Jobs are high level tasks that define one or more steps that are needed to be run for the job to complete. The actual work done by a job takes place in steps. Each workflow contains one or more jobs.
  -   Steps are simple commands, shell scripts, or they can be actions. Actions help speed up your workflow development by bundling programs, runtimes and all of their dependencies into docker containers or JavaScript modules. The actual work done by a job takes place in steps.
  -   Actions help speed up your workflow development by bundling programs, runtimes and all of their dependencies into docker containers or JavaScript modules. Actions are useful for running common tasks in a repeatable fashion without having to write the same code over and over again.
  -   Matrix strategy in a GitHub Actions workflow allows for the automatic creation of copies of the workflow, each running the same steps with different values for the specified variable.

## CI/CD Software Packages
-  Configure the package registry
-  Authenticate with the registry
-  Build the package
-  Publish the package to the registry
-  Each language has a specific configuration that identifies the target registry and how to authenticate with it. (Java=> settings.xml and pom.xml, JS=> package.json, Ruby=> .gemspec, .Net=> .csproj)
