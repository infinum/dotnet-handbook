## .gitignore

.[gitignore](https://git-scm.com/docs/gitignore) file tells Git which files in your project should be ignored. It is usually (most often) located in the root directory of your project. File content example for .Net Core project can be found [here](https://github.com/dotnet/core/blob/master/.gitignore).
 Be aware that adding a certain path to .gitignore will have no effect if the file is already being tracked by Git. In this case you can remove the file an then update the .gitignore file.

 ## GitFlow

GitFlow is a branching model for Git which we use here at Infinum. Its main advantage is that it supports projects where deployments are made often. Since we use agile workflow, this goes hand in hand with it. We will cover some of things about the GitFlow in this handbook, but you can find a broader explanation [here](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).

This image shows the git flow branches diagram:



<img src="C:\Infinum\Handbook .NET\git-flow.png" alt="gitflowimage" style="zoom:100%;" />



### master

Master branch should only contain a release ready code and is one of the two branches we are releasing the code from.

### develop

Develop branch is a second branch which we release code from (for development environment). It should only contain the code which is ready to be part of the next release. When it is decided that a new release build should be done a release branch is created from the develop branch. After a successful deployment, we merge the release branch into master and back into develop.

### feature

Each new feature needs to be implemented in a separate branch. This is always branched off from the develop branch and then merged back into it. Feature branches also have a naming convention we should follow: **feature/feature-name**. If the project is using a feature tracking system, then it is necessary to put the ticket identifier into the name as well: **feature/T-1234-feature-name**.

### hotfix

If we recognize a bug or important change request that needs to be resolved on production as soon as possible, we can create a hotfix branch. Hotfix is branched off the master branched and then merged back into master and develop branch. We use standard hotfix naming convention which is similar to the feature one: **hotfix/T-3456-name**.