## .gitignore

.[gitignore](https://git-scm.com/docs/gitignore) file tells Git which files in your project should be ignored. It is usually (most often) located in the root directory of your project. A file content example for the .NET Core project can be found [here](https://github.com/dotnet/core/blob/master/.gitignore).

Be aware that adding a certain path to .gitignore will have no effect if the file is already being tracked by Git. In this case, you can remove the file and then update the .gitignore file.

## Git Tags

**Git tag** is a point of reference to a specific commit. Usually, it's used to add a mark with a current release version. Tags can also help automate, orchestrate, and monitor development processes in a CI/CD pipeline.

There are two types of git tags:
1. **Lightweight tag** - a pointer to a specific commit,
2. **Annotation tag** - tag with a message, creation date, tag author's name and e-mail and a checksum; stored as an object in the Git database and made available for searching

If a good branching strategy with clearly named release branches is used, tags can be made obsolete, unless they are not required by the team's CI/CD policy.