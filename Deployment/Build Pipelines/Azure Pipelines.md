## Azure DevOps

To understand **Azure Pipelines** and use them more efficiently, here is the [link](https://learn.microsoft.com/en-us/azure/devops/pipelines/get-started/key-pipelines-concepts?view=azure-devops) to the documentation for the key concepts. There is a short video and documentation that explain the basic terms and parts of a pipeline in detail.

### Manual triggers

Here are 5 easy steps that explain how to run a pipeline manually:

1. Go to Azure DevOps and select your project
2. Click on **Pipelines** in the sidebar
	![azurepipelinessection](/resources/azure-pipelines-section.png)
3. Click on pipeline that you want to run (e.g. `pipelines-dotnet-core`)
4. Click on the blue **Run Pipeline** button and a sidebar will pop up
	![azurepipelinesrunpipeline](/resources/azure-pipelines-run-pipeline.png)
5. In the **Run Pipeline** dialog, select the branch and click Run. Azure Devops will queue the job and start the process.

For more info on manual triggers you can visit the [documentation](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/stages?view=azure-devops&tabs=yaml#add-a-manual-trigger) on manual triggers

## Automatic triggers

