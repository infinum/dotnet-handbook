## Branching strategies

**Branching strategy** is a set of rules created by developers used to achieve a consistent team-level approach to the management of git branches within a project.

There are many possible branching strategies. Good branching strategies follow two basic rules:

1. Reduce the number of merge conflicts (main cause: changes of the same resources by multiple authors and long-living branches)
2. Allow deployments based on the client and project needs (main goal)


### Good branch naming

When creating a new branch, name it consistently to the team's branching strategy. First, determine the type of the branch (release, feature, hotfix). Second, describe the purpose of creating a branch using not more than a couple of words. Following are some examples of branch names according to these rules:

- `feature/add-user-basic-authentication`
- `feature/task-344096-add-download-report-endpoint`
- `release/v3.1.1`
- `hotfix/resolve-user-account-creation-bug`


### Examples of branching strategies

#### GitFlow

GitFlow is one of the most popular branching models used commonly within the .NET team. Its main advantage is that it supports projects where deployments are often. Since we use agile workflow, this goes hand in hand with it. We will cover some of the things about GitFlow in this handbook, but you can find a broader explanation [here](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

This image shows the diagram of GitFlow branches:

![gitflowimage](/resources/git-flow.png)

##### master/main

The `master` branch (sometimes called `main`) should only contain a release-ready code and is one of the two branches from which code is released.

##### develop

The `develop` branch is the second branch from which the code is released (for the development environment). It should only contain the code which is ready to be part of the next release. When it is decided that a new release build should be done, a release branch is created from the develop branch. After a successful deployment, we merge the release branch into the `master` and back to the `develop` branch.


##### feature

Each new feature needs to be implemented in a separate branch. It is always branched off from the `develop` branch and merged back into it. Feature branches also have a naming convention we should follow: 

```feature/feature-name```


If the project uses a feature tracking system, it is necessary to put the ticket identifier into the name as well:

```feature/T-1234-feature-name```


##### hotfix

If there is a bug or a priority request for a change that needs resolution in production as soon as possible, we can create a hotfix branch. A hotfix is branched off the `master` branch and merged back into it and `develop`. We use a standard hotfix naming convention similar to the feature one:

```hotfix/bug-name```

```hotfix/T-3456-name```


##### Pros and cons

##### Pros

- allows handling multiple versions of the same code (TEST, UAT, PROD...)


##### Cons

- complicated merging sequence
- releases can be tricky (if branches don't merge correctly, changes need to be chosen one by one (cherry-picked))
- a lot of long-living branches (`develop`, `master`, release branches)


#### GitHub flow

GitHub flow is a simple flow having only one long-living branch (`master`/`main`) from which feature branches originate. After work on a feature branch completes, changes are merged to the `master` branch using a pull request.

![githubflowimage](/resources/github-flow.png)

[(image source)](https://media.geeksforgeeks.org/wp-content/uploads/20220214111138/GitHubFlow.jpg)


##### Pros

- reduced number of long-living branches
- ideal for CI/CD (allows deployment of all the features as soon as they are merged)
- suitable for usage in smaller projects and by smaller teams


##### Cons

- no direct way to have multiple versions of the code (achievable by using workarounds)
- more susceptible to bugs since all features get released


#### GitLab flow

GitLab flow introduces environment branches to GitHub flow (e.g., staging and production).


![gitlabflowimage](/resources/gitlab-flow.png)


The GitLab flow is based on 11 rules:
1. Use feature branches, not direct commits on master.
2. Test all commits, not only ones on master.
3. Run all the tests on all commits (if your tests run longer than 5 minutes, have them run in parallel).
4. Perform code reviews before merging into master, not afterward.
5. Deployments are automatic, based on branches or tags.
6. Tags are set by the user, not by CI.
7. Releases are based on tags.
8. Pushed commits are never rebased.
9. Everyone starts with `master` and targets `master`.
10. Fix bugs in `master` first and release branches second.
11. Commit messages reflect intent.


##### Pros

- supports multiple versions of the code (multiple pre-production environments)

##### Cons

- can be difficult to control which features go to production (e.g., code running fine in development may not work so fine in pre-production environments, so changes need to be cherry-picked)


#### Trunk based development

This branching strategy is explained in detail [here](https://trunkbaseddevelopment.com/).

##### Basic rules

- commit frequently, at least once per day
- merge branches as soon as possible


##### Pros

- no long-living branches except master
- good for CI/CD
- good visibility of code changes made by others
- (almost) always working on the newest code


##### Cons

- The longer the team is working together, the better (requires a certain level of consistency in the practices of team members)
- doesn't fully support pull requests (review of pull requests takes some time)

### Summary

Previously shown branching strategies are just examples of best practices that may or may not fit the needs of a specific project. Therefore, choosing the one that works the best should be done by following the already mentioned good branching strategy rules and taking into account the professional experience of team members and team processes.

You can learn more about git branching by using [interactive web tools like this one](https://learngitbranching.js.org/) or by reading [this Medium article by Patrick Porto](https://medium.com/@patrickporto/4-branching-workflows-for-git-30d0aaee7bf) (used as a source for this article).
