## Build pipelines

A **build pipeline** is an automated workflow that takes your code from source control, compiles it, and prepares it for deployment. It ensures consistency, automates repetitive tasks, and catches issues early, enabling faster and more reliable software delivery. Build pipelines are typically configured using YAML files, where you define the steps and conditions required for your project.

## Tools for Build Pipelines

We primarly use **GitHub Actions**, a tool for managing build and deployment pipelines. Other tools are **Azure DevOps**, **GitLab**...

The repository that contains our examples of build pipelines can be found [here](https://github.com/infinum/dotnet-pipeline-templates). 

## Manual Triggers in Build Pipelines

Manual triggers in build pipelines allow developers to run a pipeline on demand instead of automatically triggering it based on events (e.g., new commits or pull requests). This feature provides flexibility for scenarios where automation isn't always desired or when specific conditions must be met before initiating the pipeline.

### When to Use Manual Triggers

* When testing a pipeline configuration or experiment with changes before committing
* Deploy to sensitive environments like staging or production
* For urgent fixes or specific branches that don’t have automatic triggers configured

## Automatic triggers

