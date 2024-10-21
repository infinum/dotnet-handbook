## PR templates

Pull request templates provide a simple and effective way to specify how the description of a pull request should be structured and what details it should include. The template's author defines the skeleton, and the author of the pull request needs to fill in the blanks following the instructions written by the template's author.

The pull request template may include the following sections:

- Description (what the PR does)
- Type of change (bug fix, new feature, etc.)
- Link to the ticket (task, issue, etc.)
- Checklist (were the tests added for the change, does the documentation need to be updated, does the change affect the application configuration and settings, etc.)

### Adding a pull request template in GitHub

1. Create a .github folder in the root folder of your project's solution (if it doesn't already exist)
2. In the folder, create a new file with the extension ".md" and name it as you wish (e.g. "pull_request_template.md")
3. Add a pull request template to the file; the following is an example of a pull request template used in our team:

```
## Ticket [^1]

[#000 - Ticket name](https://link-to-ticket.com)

## Description [^2]

* Description of the changes made

## Checklist [^3]

- [ ] I have added tests to cover my changes
- [ ] My change requires a change to the documentation
- [ ] My change requires a change to the app settings

[^1]: Provide a link to the ticket.
[^2]: Provide a short description and/or list of changes. The idea is to tell the reviewer what they should focus on and if there are some details they should be aware of.
[^3]: Select the ones that apply to this PR.
```

4. Save the file, commit it, and push it to the main branch of your repository (create a pull request if the policy of your team does not allow direct pushes to the main branch)

After the changes are added to the main branch, the PR template will automatically apply to every new pull request created in the repository.

For adding the pull request template via Github UI, please check [the instructions in the following article on Github's official website.](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)

### Adding a pull request template in Azure DevOps

1. Use the root folder of your project's solution or create one of the following folders in the root folder of your project's solution (if none of them already exists):

- ".azuredevops"
- ".vsts" 
- "docs"

2. Create a new pull request template file according to the following rules:

- To add a PR template that will be default for all new pull requests in the solution, add a new file to the folder from step 1 with the extension ".md" or ".txt" and name it as you wish; e.g: "pull_request_template.md"
- To add a PR template that will be default for new pull requests targeting a specific branch or set of branches:
	- In the folder from step 1, create a new folder structure: `pull_request_template/branches/`
	- Add a new file and name it according to the name of the target branch or set of branches:
		- If a branch is named "feature/branch1", the file should be called "feature-branch1.md" or "feature-branch1.txt"
		- If the target set of branches is the set of all feature branches, the file should be named "feature.md" or "feature.txt"
		- If the target branches are "feature/branch1" and "feature/branch2", the file should be named "feature-branch.md" or "feature-branch.txt" (the file name without an extension should be a common prefix of the names of the target branches)"
- To add a PR template that can be appended to the PR description on demand (when creating the PR):
	- In the folder from step 1, create a new folder structure: `pull_request_template/`
	- Add a new file and name it as you wish; e.g: "additional_pr_template.md" or "additional_pr_template.txt"

3. Add a pull request template to the file; the following is an example of a pull request template used in our team:

```
## Ticket [^1]

[#000 - Ticket name](https://link-to-ticket.com)

## Description [^2]

* Description of the changes made

## Checklist [^3]

- [ ] I have added tests to cover my changes
- [ ] My change requires a change to the documentation
- [ ] My change requires a change to the app settings

[^1]: Provide a link to the ticket.
[^2]: Provide a short description and/or list of changes. The idea is to tell the reviewer what they should focus on and if there are some details they should be aware of.
[^3]: Select the ones that apply to this PR.
```

4. Save the file, commit it, and push it to the main branch of your repository (create a pull request if the policy of your team does not allow direct pushes to the main branch)

After the changes are added to the main branch, the PR template will be available for new pull requests depending on its type:

- Branch-specific PR template will be applied automatically when a new PR is created targeting the branch whose name has a prefix that is equal to the name of the template
	- If multiple PR templates can be applied to the same PR, the template with the longest name will be applied
	
- General PR templates will be applied automatically if no branch-specific PR templates have not been set already
- Additional PR templates can be appended to the PR description manually when creating the PR in the DevOps UI
	- NOTE: If there are multiple PR templates available, it's possible to append an arbitrary number of them to the PR description. This also includes the default/general and branch-specific PR templates that have not been applied by default.

For more details on the topic, please check [the following article on the Microsoft's official website.](https://learn.microsoft.com/en-us/azure/devops/repos/git/pull-request-templates?view=azure-devops)
