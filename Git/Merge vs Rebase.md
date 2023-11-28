## Merge vs. rebase

### Merge

Let there be a branch `feature1` which originates from branch `develop`. After creating the branch `feature1`, new commits have been added to both branches. If an author of branch `feature1` wants to bring new changes from `develop` to its branch, he/she can use the git `merge` command to achieve that:

```
**git checkout feature**
**git merge develop**
```


or written in one line:

```
**git merge feature1 develop**
```

 
The `merge` command creates a new, single commit in branch `feature1` which appends the history of newly added commits from branch `develop` to the history of `feature1`.


![gitmergeimage](/resources/git-merge.png)

[(image source)](https://www.atlassian.com/git/tutorials/using-branches/git-merge)

 
This way, both branches are preserved with the cost of the additional merge commit, which can be an issue if there are many branches and merging is frequent.


### Rebase

For the purpose described above, the author of branch `feature1` can also use the git `rebase` command:


```
**git checkout feature1**
**git rebase develop**
```


The `rebase` command changes the base of the `feature1`. It splits from `develop` in the latest commit instead of the original one. This way, branching is replaced by a linear structure, effectively making `feature1` a continuation of the `develop` branch. 


![gitrebaseimage](/resources/git-rebase.png) 

[(image source)](https://www.atlassian.com/git/tutorials/rewriting-history/git-rebase)


Even though this approach simplifies the project git tree structure and navigation, using it can lead to code synchronization issues if the public branches (`main`, `develop`) are rebased to private branches (`feature`, `hotfix`). Those synchronization issues can be resolved by additional merges, resulting in unnecessary commit duplication and can complicate branch management a lot.


### Summary

For more details about `merge` and `rebase` commands, please check [this article about merging and rebasing on the Atlassian website](https://www.atlassian.com/git/tutorials/merging-vs-rebasing).
