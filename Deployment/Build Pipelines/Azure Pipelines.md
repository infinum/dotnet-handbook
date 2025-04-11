## Azure DevOps

Azure DevOps is a suite of development tools provided by Microsoft, and Azure Pipelines is a powerful CI/CD service within it. It enables you to automate your builds, tests, and deployments across various platforms.

To understand **Azure Pipelines** and use them more efficiently, visit the [Key Pipelines Concepts](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops). There is a short video and documentation that explain the basic terms and parts of a pipeline in detail.

## Triggers

Azure Pipelines supports automatic triggers to run pipelines based on specific events, such as code pushes, pull requests, or scheduled times. This ensures that CI/CD processes integrate seamlessly into the development workflow. You can define triggers for branches, tags, or schedules to suit the CI/CD requirements. For more information about triggers visit the [Azure Pipelines documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/build/triggers?view=azure-devops).

### How do you build the build pipeline? 

You can add the YAML file 'manually', by committing it in your repository or through Azure DevOps. Here is an example of how to add a YAML file in your repository through Azure DevOps:

#### 1. Navigate to the **Pipelines** tab in your Azure DevOps repository

Click on the **New pipeline** button in the top right corner. Then you will be prompted to select your repository and type of YAML template to use, or just select any template and write your own.

#### 2. Write the YAML file and save the changes

In this example, we used a custom YAML file for building, testing, and deploying ***Example.Api*** project:

```yaml
variables:
- name: vmImageName
  value: 'ubuntu-latest'
- name: workingDirectoryApi
  value: '$(System.DefaultWorkingDirectory)/src/Example.Api' # your app directory
- name: testDirectoryApi
  value: '$(System.DefaultWorkingDirectory)/src/Example.Api.Tests' # your test directory
- name: dotNetRuntime
  value: '8.0.x'
- name: RuntimeStack
  value: 'DOTNETCORE|8.0'
- name: publishApiArtifactName
  value: 'aptifact'
- name: azureSubscription
  value: '_Your_Subscription_' # your Azure subscription
- name: environment
  value: 'development' # your env
- name: appServiceName
  value: 'example-api' # your app service name on Azure Portal
  
# Define pipeline-level pool
pool:
  vmImage: $(vmImageName)

# Triggers
trigger:
  branches:
    include:
      - master # your branch(es)


# Stages
stages:
- stage: Build
  displayName: Build the app

  jobs:
  - job: Build
    displayName: Building

    steps:
    - task: UseDotNet@2
      inputs:
        version: $(dotNetRuntime)
        packageType: runtime

    - task: DotNetCoreCLI@2
      displayName: Build api
      inputs:
        command: 'build'
        projects: |
          $(workingDirectoryApi)/*.csproj
  
    - task: DotNetCoreCLI@2
      displayName: 'Test api'
      inputs:
        command: 'test'
        projects: '$(testDirectoryApi)/*.csproj'

    - task: DotNetCoreCLI@2
      displayName: 'Publish Api'
      inputs:
        command: publish
        publishWebProjects: false
        projects: '$(workingDirectoryApi)/*.csproj'
        arguments: '--output $(Build.ArtifactStagingDirectory)/api'
        zipAfterPublish: True

    - task: PublishBuildArtifacts@1
      inputs:
        pathtoPublish: '$(Build.ArtifactStagingDirectory)/api'
        artifactName: '$(publishApiArtifactName)'

- stage: Deploy
  displayName: Deploy stage
  dependsOn: Build
  condition: succeeded() # build must be successful for the Deploy stage to start

  jobs:
  - deployment: Deploy
    displayName: Deploy
    environment: $(environment)
    strategy:
      runOnce:
        deploy:
          steps:

          - task: AzureRmWebAppDeployment@4
            inputs:
              ConnectionType: 'AzureRM'
              azureSubscription: '$(azureSubscription)'
              appType: 'webAppLinux'
              WebAppName: $(appServiceName)
              packageForLinux: '$(Pipeline.Workspace)/$(publishApiArtifactName)/*.zip'
              RuntimeStack: '$(RuntimeStack)'
```

After adding the YAML file, save it, and you are ready to run the pipeline.

## Manually run the pipeline

Manual execution is useful for different cases, e.g., testing pipeline changes, running on a non-triggered branch, or deploying a hotfix. Here are the steps that explain how to run a pipeline manually:

#### 1. Navigate on **Pipelines** tab

![azurepipelinessection](/resources/azure-pipelines-section.png)

Select the pipeline, e.g., `pipelines-dotnet-core`. 

#### 2. Click on the blue **Run Pipeline** button, and a sidebar will pop up.

![azurepipelinesrunpipeline](/resources/azure-pipelines-run-pipeline.png)

In the sidebar, select the branch and click **Run**. Azure DevOps will queue the job and start the process.

## Logs

If the pipeline run fails, you can inspect the logs by clicking on the run. The run details will show up, along with errors and warnings. You can even see which stage and job failed.

To access the logs, click on the error.

**Pro Tip:** Use the search bar (`Ctrl + F`) in the logs view to find specific errors or warnings.

For more details, visit the [Azure DevOps documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/troubleshooting/review-logs?view=azure-devops&tabs=windows-agent).