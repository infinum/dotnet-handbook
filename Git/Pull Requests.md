## Pull requests

Definition and basic rules of a good PR can be found [here](https://infinum.com/handbook/dotnet/team-practices/new-team-members#prs).
	
There could also be additional pre-defined rules that every PR needs to satisfy before getting an approval (all required reviewers have approved the PR, all comments have been resolved, etc.). On top of that, all merge conflicts (concurrent changes of the same resources) should be resolved before the merge. 

After merging, all changes from the source branch are available in (appended to) the target branch. Its pointer changes to point to the last commit of the pull request. Now, the source branch is obsolete and can be removed.

### How to write a good PR description
	
Every PR should have a clear and a meaningful description of the feature it introduces. Following is an example of a good PR description:

```
### Vulnerable dependencies chapter	// Name of the PR (explanation of a feature)

### Add a new chapter 'Vulnerable dependencies' with information about a way of handling vulnerable packages.	// Description of the change

### Handbook chapter 'Security'		// What is changed

### Task https://tasks.com/123		// Additional comment(s), links to related task(s)...
```

### PR review

Before merging a PR, it's important to do a good and thorough PR review. Guidelines can be found [here](https://infinum.com/handbook/dotnet/team-practices/mentors#pr-reviews).
