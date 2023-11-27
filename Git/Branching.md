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
