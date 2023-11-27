## Pull requests

Definition and basic rules of a good PR can be found [here](https://infinum.com/handbook/dotnet/team-practices/new-team-members#prs).
	
There could also be additional pre-defined rules that every PR needs to satisfy before getting an approval to be merged (all required reviewers have approved the PR, all comments have been resolved, etc.). Also, all potential merge conflicts (concurrent changes of the same resources) should be resolved prior to the merge. 

After merging, all changes from the source branch are available in (appended to) the target branch with its pointer adjusted to the last commit of the pull request. At this stage, it's also recommended to delete the source branch as it won't be used anymore.

### How to write a good PR description
	
Every PR should be accompanied by a clear and meaningful description of the feature being introduced by that PR. Following is an example of a good PR description:

```
### Vulnerable dependencies chapter	// Name of the PR (explanation of a feature)

### Add a new chapter 'Vulnerable dependencies' with information how to handle vulnerable packages.	// Description of the change being introduced

### Handbook chapter 'Security'		// What is being changed

### Task https://tasks.com/123		// Additional comment(s), links to related task(s)...
```

### PR review

Before merging a PR, it's important to do a good and thorough PR review. Guidelines can be found [here](https://infinum.com/handbook/dotnet/team-practices/mentors#pr-reviews).
