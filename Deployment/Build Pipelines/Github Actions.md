## GitHub Actions

Github uses a bit different terminology from Azure, but in practice, it is similar. The [documentation](https://docs.github.com/en/actions/writing-workflows/about-workflows#workflow-basics) explains the basics.

GitHub provides preconfigured workflow templates that can be used as is or customized to create your workflow. Also, GitHub analyzes the code and shows the workflow templates that might be useful for your repository (e.g., we will see suggestions for .NET projects).

### Manual triggers

The manual triggering is accomplished using the `workflow_dispatch` event in the `on` section of the YAML file, which creates a **"Run workflow"** button in the **Actions** tab of the repository when the workflow is selected.

You can add the YAML file 'manually' by committing it to your repository or through GitHub. The following is an example of how to add a YAML file to the repository through GitHub.

#### 1. Open the **Actions** tab in your GitHub repository

You can browse the templates and select the desired one by clicking the *"Configure"* button.

![githubaction](/resources/github-actions.png)

#### 2. Write the YAML file and commit changes

In this example, *"Deploy a .NET Core app to an Azure Web App"* is used with a manual trigger:

```yaml
name: Build and deploy ASP.Net Core app to an Azure Web App

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

After committing the changes to the repository, you will see this in the **Actions** tab:

![githubrunworkflow](/resources/github-actions-run-workflow.png)

Now, you can easily test the workflow by running it by clicking *"Run workflow"* button.

## Automatic triggers

Automatic triggers in GitHub Actions allow workflows to run automatically based on specific events in your repository. These triggers are defined in the `on` section of the workflow YAML file. Frequent use cases include triggering builds when pushing the code, creating pull requests, or meeting schedules.

Here's how to set up automatic triggers in a workflow:

#### Define the trigger event

To enable automatic triggers, replace or extend the `on` section of the YAML file with one or more supported GitHub events. Examples of common automatic triggers are:

- Push events: Trigger the workflow when pushing the code to a specific branch or tag (e.g., the `develop` branch):

    ```yaml
    # name and env sections

    on:
      pull_request:
        branches:
          - develop

    # job sections for build, test and deploy
    ```

- Pull requests: Trigger the workflow when a pull request is opened, synchronized, or closed.

    ```yaml
    # name and env sections

    on:
      push:
        branches:
          - develop

    # job sections for build, test and deploy
    ```

- Scheduled workflows: Use `schedule` to trigger workflows at specific times using cron syntax.

    ```yaml
    # name and env sections

    on:
      schedule:
        cron: '0 0 * * *' # Runs at midnight UTC every day

    # job sections for build, test and deploy
    ```

- Other events: There are many other events, such as `release`, `issues`, or `workflow_run`. See the complete list of events in [GitHub documentation](https://docs.github.com/en/actions/writing-workflows/choosing-when-your-workflow-runs/events-that-trigger-workflows).

You can combine multiple events in the `on` section of a GitHub Actions workflow, both automatic and manual. This approach allows the workflow to trigger different event types, making it highly flexible.

Benefits of automatic triggers:
- **Continuous Integration (CI):** Ensure code changes are tested and validated before merging.
- **Time-based automation:** Use scheduled workflows for tasks like backups, reporting, or maintenance scripts.
- **Scalability:** Automate repetitive tasks, reducing manual effort and human error.

## Logs 

If the workflow fails, the logs can be inspected by clicking on the workflow run. The workflow graph shows up, and you can see which step failed and the error messages in the logs.

![githublogs](/resources/github-actions-error-workflow.png)

![githublogs](/resources/github-actions-error-logs.png)
