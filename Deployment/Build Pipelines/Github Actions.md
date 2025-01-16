## GitHub Actions

Github uses a bit different terminilogy from Azure, but in practice it is similar. Here is the link to [documentation](https://docs.github.com/en/actions/writing-workflows/about-workflows#workflow-basics) where basics are explained.
GitHub provides preconfigured workflow templates that you can use as-is or customize to create your own workflow. Also, GitHub analyzes your code and shows you workflow templates that might be useful for your repository (e.g. we will see suggestions for .NET projects).

### Manual triggers

Manual trigger is accomplished using the `workflow_dispatch` event in YAML file, which creates a **"Run workflow"** button in the **Actions** tab of your repository when you select your workflow.

Here is an example how to add a YAML file in your repository.

#### 1. Open the **Actions** tab in your GitHub repository

You can browse the templates and select the desired one by clicking on *"Configure"* button. In this example, we used *"Deploy a .NET Core app to an Azure Web App"*

![githubaction](/resources/github-actions.png)

#### 2. Write a file name and content, then commit the changes

In this example, we used an auto-generated file by GitHub:

```yamlname: Build and deploy ASP.Net Core app to an Azure Web App

env:
  AZURE_WEBAPP_NAME: example-app-name    # set this to the name of your Azure Web App
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '8'                 # set this to the .NET Core version to use

on:
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Set up .NET Core
        uses: actions/setup-dotnet@v4
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: dotnet publish
        run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/example-app

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: .net-app
          path: ${{env.DOTNET_ROOT}}/example-app

  deploy:
    permissions:
      contents: none
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: 'Development'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}

    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: .net-app

      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: ${{ env.AZURE_WEBAPP_NAME }}
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

![githubworkflowexample](/resources/github-actions-workflow-example.png)

GitHub even displays the documentation for workflows to help you configure the workflow.

#### 3. Run the workflow

After you have commited the changes to your repository, you will see this in the **Actions** tab:

![githubrunworkflow](/resources/github-actions-run-workflow.png)

Now, you can easily test your workflow by running it with the click on *"Run workflow"* button.

#### Logs 

If the workflow fails, you can inspect the logs by clicking on the workflow run. The workflow graph will be displayed, and you can see which step failed and the error messages in the logs.

![githublogs](/resources/github-actions-error-workflow.png)

![githublogs](/resources/github-actions-error-logs.png)

## Automatic triggers

