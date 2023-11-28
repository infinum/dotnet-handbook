## .gitignore

.[gitignore](https://git-scm.com/docs/gitignore) file tells Git which files in the project to ignore. Most often, it is in the root directory of the project. A file content example for the .NET Core project can be found [in the .gitignore file of the .NET GitHub repository](https://github.com/dotnet/core/blob/master/.gitignore).

Be aware that adding a path to .gitignore will have no effect if Git already tracks the file. In this case, you can remove the file and update the .gitignore file.


## Commits

A Git commit is a snapshot of a code repository at a specific point in time.

### Rules for making a good commit

#### Be meaningful and consistent


- Organize changes into small groups of related changes. All groups should be committed separately (similar to the Single Responsibility Principle)
- Describe in simple terms what changes by applying the commit


#### Write a commit message using the imperative present tense


- Describe what happens if a particular commit is applied
- You're free to use past tense if that's a team decision; in general, any of the two ways of writing a message is allowed as long as it is consistently applied and fits client and project needs
	- NOTE 1: Present tense verbs tend to be shorter than past tense verbs, which makes messages written that way shorter
 	- NOTE 2: Imperative messages are more readable and applicable during a review of branch/commit before merge; past tense messages are useful when reviewing past commits


### Examples of bad commit messages


```
Fix PR comments
```

**Reason**: Unclear what exactly the commit covers.


```
Add endpoint for calculating credit score and get user credit score.
```

**Reason**: Two separate features in a single commit.


### Examples of good commit messages


```
Extend user response with user ID property.
```

**Reason**: Single standalone feature described.


```
Add test for validation of mobile phone number.

Add a test scenario to check if entering a character as a mobile number is forbidden.

Bug-123
```

**Explanation**: 1st row - subject, 2nd row - body, 3rd row - footer concerning a bug/task.

**Reason**: The commit message is written with all the necessary details provided.


## Branching

Git branches enable concurrent work on a code repository and the addition of unverified changes without affecting the code used in a working environment.

Each branch represents a series of changes beginning with some commit. It is essentially a pointer to the last commit in that series that is adjusted every time a new commit appears. Multiple branches can start in the same commit and exist in parallel.

### Branch types

Branches differ only in terms of usage (context and relevant flow). There are two branch types: **long-living** and **short-living**. 

Long-living branches include the following types:

- ``develop`` (the latest code in the development stage)
- ``release`` (code of a particular release to TEST/UAT environment)
- ``master`` (the latest working/PROD environment code)


Short-living branches include the following types:

- ``feature`` (all commits related to a single feature/task)
- ``hotfix`` (commits that fix an issue in the working/PROD environment)

## Git Tags

**Git tag** is a point of reference to a specific commit. Usually, it's used to add a mark with a current release version. Tags can also help automate, orchestrate, and monitor development processes in a CI/CD pipeline.

There are two types of git tags:

1. **Lightweight tag** - a pointer to a specific commit
2. **Annotation tag** - tag with a message, creation date, tag author's name and e-mail, and a checksum; stored as an object in the Git database and made available for searching

If a good branching strategy with named release branches is used, tags are obsolete unless the team's CI/CD policy requires so.
