## Merge vs. rebase

### Merge

Suppose there is a branch feature1 which originates from branch develop. After creation of the branch feature1, new commits have been added to the both branches. If an author of branch feature1 wants to bring new changes from develop to its branch, he/she can use git **merge** command to achieve that:
- **git checkout feature**
- **git merge develop**

or written in one line:
- **git merge feature1 develop**
 
Merge command would create a new, single commit in branch feature1 which would append the history of newly added commits from branch develop to the history of feature1.

![gitmergeimage](/resources/git-merge.png) [(image source)](https://www.atlassian.com/git/tutorials/using-branches/git-merge)
 
This way, both branches are preserved with the cost of the additional merge commit, which can be an issue if there are many branches created and merging is done frequently.

### Rebase

For the purpose described above, author of branch feature1 can also use git **rebase** command:
- **git checkout feature1**
- **git rebase develop**

Rebase command would change a base of the feature1 - it would split from develop in the latest commit instead of the original one. This way, branching would be replaced by a linear structure, effectively making feature1 a continuation of develop branch. 

![gitrebaseimage](/resources/git-rebase.png) [(image source)](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)

Even though this approach simplifies the project git tree structure and navigation, using it can lead to code synchronization issues if the public branches (main, develop) are rebased to private branches (feature, hotfix). Those synchronization issues can be resolved by additional merges which adds unnecessary commits duplication and can complicate branch management a lot.

### Summary

For more details about merge and rebase commands, you can also check [here](https://www.atlassian.com/git/tutorials/merging-vs-rebasing).