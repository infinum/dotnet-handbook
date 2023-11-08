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
