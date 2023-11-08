## .gitignore

.[gitignore](https://git-scm.com/docs/gitignore) file tells Git which files in your project should be ignored. It is usually (most often) located in the root directory of your project. A file content example for the .NET Core project can be found [here](https://github.com/dotnet/core/blob/master/.gitignore).
 Be aware that adding a certain path to .gitignore will have no effect if the file is already being tracked by Git. In this case, you can remove the file and then update the .gitignore file.

## Commits

Git commit is a snapshot of a code repository at a specific point in time.

Good commit should follow some basic rules:
1. **Be meaningful and consistent**.
	- Changes should be organized into as small groups of related changes as possible and all the groups should be committed separately (similar to Single Responsibility Principle)
	- Author should be able to describe in simple terms what change will be introduced by applying the commit
2. **Write a commit message using imperative present tense**.
   	- Describes what happens if/when a particular commit is applied
	- Past tense can also be used if that's a team's decision; in general, any of the two ways of writing a message is allowed as long as it is consistently applied and fits client and project needs
	- NOTE 1: Present tense verbs tend to be shorter than past tense verbs, which makes messages written that way shorter
	- NOTE 2: Imperative messages are more readable and applicable when branch/commit is reviewed before it's merged; past tense messages can be useful when reviewing past commits

### Examples of good and bad commit messages

Examples of bad commit messages:
1. message: Fix PR comments
	- Explanation: Unclear what was done
2. message: Add endpoint for calculating credit score and get user credit score
	- Explanation: Two separate features in a single commit

Examples of good commit messages:
1. message: Extend user response with user ID property
	- Explanation: Single standalone feature described
2. message:

	Add test for validation of mobile phone number

	Add test scenario to check if entering character as a mobile number is forbiden

	Bug-123
	- Explanation: 1st row - subject, 2nd row - body, 3rd row - footer with a reference to a bug/task

## Branching

To enable concurrent work on a code repository or adding unverified changes without affecting the code used in a working environment, git **branches** are used.

Each branch represents a series of changes beginning with some commit. It is essentially a pointer to a last commit in that series (adjusted each time when a new commit is created). Multiple branches can start in a same commit and exist in parallel. All the branches can be seen as a logical git tree.

### Branch types

Branches differ only in terms of how they are used (context and relevant flow). There are two main types of branches: **long-living** and **short-living** branches. 

Long-living branches include following types:
- develop (the latest code in development stage)
- release (code of a particular release to TEST/UAT environment)
- master (the latest working/PROD environment code)

Short-living branches include following types:
- feature (all commits related to a single feature/task)
- hotfix (commits which fix an issue in working/PROD environment)

## Branching strategies

**Branching strategy** is a set of rules created by developers used to achieve consistent team-level approach to creation and management of git branches within a project. 

There are many possible branching strategies. Good branching strategies follow two basic rules:
1. Reduce number of merge conflicts (main causes: changes of the same resources by multiple authors and long-living branches)
2. Allow deployments based on the client and project needs (main goal)

### Good branch naming

When creating a new branch, author should pay attention to name it consistently to team's branching strategy. First, the type of the branch should be determined (release, feature, hotfix). Second, the purpose of creating a branch should be described clearly in a couple of words. Following are some examples of branch names according to these rules:
- feature/add-user-basic-authentication
- feature/task-344096-add-download-report-endpoint
- release/v3.1.1
- hotfix/resolve-user-account-creation-bug

### Examples of branching strategies

#### GitFlow

GitFlow is a branching model for Git which we use here at Infinum. Its main advantage is that it supports projects where deployments are made often. Since we use agile workflow, this goes hand in hand with it. We will cover some of the things about GitFlow in this handbook, but you can find a broader explanation [here](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

This image shows the git flow branches diagram:

![gitflowimage](/resources/git-flow.png)

##### master

The master branch should only contain a release-ready code and is one of the two branches we are releasing the code from.

##### develop

Develop branch is the second branch which we release code from (for the development environment). It should only contain the code which is ready to be part of the next release. When it is decided that a new release build should be done a release branch is created from the develop branch. After a successful deployment, we merge the release branch into the master and back into the develop.

##### feature

Each new feature needs to be implemented in a separate branch. This is always branched off from the develop branch and then merged back into it. Feature branches also have a naming convention we should follow: **feature/feature-name**. If the project is using a feature tracking system, then it is necessary to put the ticket identifier into the name as well: **feature/T-1234-feature-name**.

##### hotfix

If we recognize a bug or important change request that needs to be resolved on production as soon as possible, we can create a hotfix branch. A hotfix is branched off the master branch and then merged back into master and develop branches. We use a standard hotfix naming convention which is similar to the feature one: **hotfix/T-3456-name**.

##### Pros and cons

**Pros**: 
- allows handling multiple versions of the same code (TEST, UAT, PROD...)

**Cons**: 
- complicated merging sequence
- releases can be tricky (if branches are not merged correctly, changes need to be chosen one-by-one (cherry picked))
- a lot of long-living branches (develop, master, release branches)

#### Github flow

**Pros**: 
- reduced number of long-living branches
- ideal for CI/CD (allows all the features to be deployed as soon as they are merged)
- suitable for usage in smaller projects and by smaller teams

**Cons**: 
- no direct way to have multiple versions of the code (achievable by using workarounds)
- more susceptible to bugs since all features get released

#### GitLab flow

GitLab flow introduces environment branches to GitLab flow (e.g. staging and production).



The GitLab flow is based on 11 rules:
1. Use feature branches, no direct commits on master
2. Test all commits, not only ones on master
3. Run all the tests on all commits (if your tests run longer than 5 minutes have them run in parallel).
4. Perform code reviews before merges into master, not afterwards.
5. Deployments are automatic, based on branches or tags.
6. Tags are set by the user, not by CI.
7. Releases are based on tags.
8. Pushed commits are never rebased.
9. Everyone starts from master, and targets master.
10. Fix bugs in master first and release branches second.
11. Commit messages reflect intent.

**Pros**: 
- supports multiple versions of the code (multiple pre-production environments)
**Cons**:
- can be difficult to control which features go to production (e.g. code running fine in development may not work so fine in pre-production environments, so changes need to be cherry-picked)

#### Trunk based development

This branching strategy is explained in details [here](https://trunkbaseddevelopment.com/).

**Basic rules**:
- commit frequently, at least once per day
- merge branches as soon as possible

**Pros**: 
- no long-living branches except master
- good for CI/CD
- good visibility of code changes made by others
- (almost) always working on the newest code

**Cons**: 
- longer the team is working on together, the better (requires certain level of consistency in the practices of team members)
- doesn't fully support pull requests (pull requests take some time to be reviewed)

### Summary

Previously showed branching strategies are just examples of best practices that may or may not fully fit the needs of a specific project. Therefore, choosing the one the fits the best should be done by following the already mentioned good branching strategy rules, as well as taking into account the professional experience of team members and team processes.

You can learn more about git branching [here](https://learngitbranching.js.org/) and [here](https://medium.com/@patrickporto/4-branching-workflows-for-git-30d0aaee7bf) (used as a source for this article).

