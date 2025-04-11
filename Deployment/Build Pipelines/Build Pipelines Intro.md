## Build pipelines

A **build pipeline** is an automated workflow that takes the code from source control, compiles it and prepares it for deployment. It ensures consistency, automates repetitive tasks, and catches issues early, enabling faster and more reliable software delivery. Build pipelines are typically configured using YAML files, where you define the steps and conditions required for your project.

## Tools for Build Pipelines

We primarily use **GitHub Actions**, a tool for managing build and deployment pipelines. Other tools are **Azure DevOps**, **GitLab**, etc.

The repository that contains our examples of build pipelines can be found [here](https://github.com/infinum/dotnet-pipeline-templates). 

## Manual Triggers in Build Pipelines

Manual triggers in build pipelines allow developers to run a pipeline on demand instead of automatically triggering it based on events (e.g., new commits or pull requests). This feature provides flexibility for scenarios where automation isn't always wanted or when specific conditions must be met before initiating the pipeline.

### When to Use Manual Triggers

* When testing a pipeline configuration or experiment with changes before committing
* Deploy to sensitive environments like staging or production
* For urgent fixes or specific branches that don't have automatic triggers configured

## Automatic triggers

Automatic triggers are an essential part of modern build pipelines. They enable workflows to execute automatically in response to specific events or conditions in a repository. These triggers eliminate the need for manual intervention and ensure that critical processes like builds, tests, and deployments happen consistently and reliably.

### Common Types of Automatic Triggers

1. **Code Changes**  
   Pipelines can be triggered automatically when pushing code changes to the repository. Example events are:
   - **Push Events**: Trigger when pushing commits to specific branches or tags.
   - **Pull Request Events**: Trigger when pull requests are opened, updated, merged, or closed.

2. **Scheduled Workflows**  
   Workflows can be configured to run at specific intervals or times using CRON expressions. This is particularly useful for running periodic tasks, like nightly builds or generating reports.

3. **Release Events**  
   Workflows can automatically run when a release is created, updated, or published in the repository, ensuring a consistent deployment process.

4. **Custom Repository Events**  
   Workflows can respond to various events such as issue creation, branch deletions, or changes to repository settings.
