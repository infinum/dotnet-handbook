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

Branches differ only in terms of how they are used (context and relevant flow). There are two main types of branches: **long-living** and **short-living** branches. 

Long-living branches include following types:
- develop (the latest code in development stage)
- release (code of a particular release to TEST/UAT environment)
- master (the latest working/PROD environment code)

Short-living branches include following types:
- feature (all commits related to a single feature/task)
- hotfix (commits which fix an issue in working/PROD environment)


## GitFlow

GitFlow is a branching model for Git which we use here at Infinum. Its main advantage is that it supports projects where deployments are made often. Since we use agile workflow, this goes hand in hand with it. We will cover some of the things about GitFlow in this handbook, but you can find a broader explanation [here](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

This image shows the git flow branches diagram:




![gitflowimage](/resources/git-flow.png)



### master

The master branch should only contain a release-ready code and is one of the two branches we are releasing the code from.

### develop

Develop branch is the second branch which we release code from (for the development environment). It should only contain the code which is ready to be part of the next release. When it is decided that a new release build should be done a release branch is created from the develop branch. After a successful deployment, we merge the release branch into the master and back into the develop.

### feature

Each new feature needs to be implemented in a separate branch. This is always branched off from the develop branch and then merged back into it. Feature branches also have a naming convention we should follow: **feature/feature-name**. If the project is using a feature tracking system, then it is necessary to put the ticket identifier into the name as well: **feature/T-1234-feature-name**.

### hotfix

If we recognize a bug or important change request that needs to be resolved on production as soon as possible, we can create a hotfix branch. A hotfix is branched off the master branch and then merged back into master and develop branches. We use a standard hotfix naming convention which is similar to the feature one: **hotfix/T-3456-name**.
