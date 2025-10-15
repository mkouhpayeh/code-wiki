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
  -  Authentication is handled by build steps
  -  Permissions are scoped using GITHUB_TOKEN
-  Authenticate with the registry
-  Build the package
-  Publish the package to the registry
  -  Package sonfigurations contain references to specific code versions
  -  Each package must use a new version number. So as a best practice, update the code to reference a new version number with each new release. Updating the version number in the package's configuration file ensures that the package being published has a distinct version that has not been used before.
-  Each language has a specific configuration that identifies the target registry and how to authenticate with it. (Java=> settings.xml and pom.xml, JS=> package.json, Ruby=> .gemspec, .Net=> .csproj)
-  If we want to become the workflow reusable and can be triggered from another workflow, such as the delivery workflow:

``` yaml
on:
  workflow_call:
```

  -  Then Copy the workflow file path by clicking the three dot option> Copy path
  -  When we want to use this workflow in another repo:

  ```
  jobs:
    integration:
       uses: username/repoName/paste the workflow path@branchName or [./path (in the same repo)]
       permissions:
         contents: read
  
  build:
     needs: [integration]
     permissions:
       contents: read
       packages: write
  ```
  
## Deploying Software
-  To implement continuous integration, we set up workflows to process our code on each push to the repo.
-  For continuous delivery, we set up workflows that were also triggered by pushes or releases. Our delivery workflows automatically created software packages or container images.
-  With continuous deployment, we'll extend this pipeline to place our code, software package, or artifact into a system where it can be used and operated, that is, each push to the repo triggers a workflow that integrates the code, creates an artifact, and then places that artifact into a system where we can use it.
-  GitHub allows us to define a deployment target as an **environment**. We can give environments names and also assign rules that govern deployments.
  -  We reference environments in a workflow by using the **environment** keyword followed by the name assigned to the environment. 
  -  We can manually create environments from the GitHub web interface or have them created automatically for us when we reference hem in a workflow.

  ```
  jobs:
    deployment:
      runs-on: ubuntu-latest
      environment: production
  ```

  -  However, creating an environment in the web interface before using it in a workflow provides the best experience for setting up other features we can use in our deployments.
  -  This includes setting up protection rules and creating variables and secrets.
  -  Environments: Protection Rules
    -  Using protection rules, we can govern access to environments based on conditions we specify.
    -  Deployment branches limit which branches in a repo can deploy to an environment. We can also use patterns to indicate which branches are considered protected.
    -  And then, we can create rules that stop a deployment until it's allowed to proceed by a reviewer or repo administrator.
  -  Environments: Variables and Secrets
    -  GitHub lets us store variables and secrets in our repositories. However, we can override these values at the environment level. Environments can have the same variables and secrets but assign different values.
    
    ```
    steps:
      -  name: Configure AWS Credentials
         uses: aws-actions/configure-aws-credentials@v2
         with:
           aws-region: ${{ vars.AWS_REGION }}
    ```
    
    -  This gives us the flexibility to create workflows and jobs that can be reused with different environments without modification. 
    -  When we reference variables in a workflow, GitHub Actions will know which value to use based on the environment. Typically, variables are used to pass configuration information or any parameters needed to run deployment commands, and we can use secrets to protect sensitive information like access keys and passwords. 
    
    ```
     steps:
       -  name: Configure AWS Credentials
          uses: aws-actions/configure-aws-credentials@v2
          with:
            aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
            aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
     ```
     
    -  These types of credentials will be important for deploying to services located outside the GitHub ecosystem. 
  -  Environments: Concurrency
    -  For deployments, we also have to consider if an environment is being used in multiple workflows. When a workflow is triggered, the default behavior is to start running as soon as possible. However, running multiple deployments may be problematic. We can control this behavior with the concurrency setting.
     
    ```
    jobs:
      deploy_staging:
         environment:Staging
         concurrency:
           group: deploy-staging
    ```
    
    -  Defining a concurrency group allows workflows to start but then automatically pause if another workflow is already deploying to the same environment. Concurrency groups can be defined at the workflow or the job context. This means we can pause entire workflows or only the jobs that might interfere with an active deployment.
