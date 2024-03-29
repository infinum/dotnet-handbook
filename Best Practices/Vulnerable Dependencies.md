The beauty of developing software using a platform that has a big community is that we don't have to implement everything ourselves. Using packages developed by other people speeds up the development process by orders of magnitude, but that also means that we're using some code we don't have control of. Because of that, we must always check if the packages we're using contain any known vulnerabilities.

## How to hunt them?

We can get the list of vulnerable packages our projects reference by executing the following command:

``` bash
dotnet list package --vulnerable --include-transitive
```

This command is used to list all the packages used by our solution. The `--vulnerable` option will check the packages for known vulnerabilities and list only those that have them. The `--include-transitive` parameter tells the command not to stop at the top level, but to go through the whole dependency tree (in other words checking dependencies of our dependencies, and then their dependencies as well).

One important note about this command is that the packages are not scanned and analyzed, but rather checked against the centralized GitHub Advisory Database. This means that any non-public packages that might not be safe will not be marked as vulnerable. You can learn more about this in Microsoft's [blog post about the command](https://devblogs.microsoft.com/nuget/how-to-scan-nuget-packages-for-security-vulnerabilities/).

## What to do about them?

Now that we've run the command, we have a list of packages with known vulnerabilities. The next step is determining whether that vulnerability is actually afflicting our system, and, if it is, how to fix it.

### Analyzing the vulnerability

This step of the process is unfortunately not straightforward. The result of the command gives us the links to the reports for each found vulnerability. The reports specify the severity and have some short descriptions of the exploit steps and what impact the vulnerability could have on a system.

The severity can be minimal, medium, high, or critical. This level is determined by a couple of factors of potential vulnerabilities: how easy is it to set up, does it require some user interaction, how much data or control can a malicious attacker obtain, etc. This level should not be directly correlated with how urgently should the issue be fixed. For example, a medium level can mean that only one user's data can be accessed, and a critical one could leak all of our users' data. While one is obviously worse than the other, we shouldn't allow either of them.

Another important thing to get from a security report is whether a certain vulnerability can be used against our system. If a vulnerability is afflicting a feature that is not being used by our solution, we don't have to be urgent when it comes to resolving the issue (e.g. a critical vulnerability found in `System.Common.Drawing` transitively used by an API which doesn't have any graphical features). There are many ways to find this out. Some packages are open-source, which enables us to analyze them and see how that package is using the vulnerable dependency. If the vulnerability afflicts the part of our solution which is not being deployed (e.g. our test tools and projects), we can safely assume that the public part of our system is not affected. But there will be situations where the available information will not be enough to come to any conclusions more meaningful than an educated guess, in which case we should proceed with fixing it.

These reports are not always easily understandable and might take a lot of time to grasp. In some cases, the impact might be obvious, but in a lot of them, the vulnerability will exploit a tiny feature in a transitive dependency which might be hard to figure out if it is being used in our solution. Because of that, it might be advisable not to go down the investigation rabbit hole and continue with the next step.

### Fixing the vulnerabilities

The process of fixing these issues is, in theory, simple: just replace all the vulnerable dependencies with newer versions that have fixed the issue, or find a replacement package that doesn't have the vulnerability. In practice, unfortunately, this is not always easy to do. Finding out which exact package we should update is not as simple as updating the packages used by our projects because we don't just check the packages referenced directly by our projects, but also the transitive dependencies. Currently, there is no native way (integrated into Visual Studio or the `dotnet` CLI tool) to find out which one of our direct dependencies uses the problematic transitive dependency. Fortunately, we weren't the first ones annoyed by this lack of support.

#### Nail Down NuGet

[NailDownNuGet](https://github.com/Kraego/NailDownNuget) is a PowerShell script that uses the [`nuget-deps-tree` package](https://www.npmjs.com/package/nuget-deps-tree) (available on `npm`) to get the dependency tree and then traverses through it to determine which package uses the transitive dependency we're searching for.

To use the script, we must first install the `nuget-deps-tree` package:

``` powershell
npm install nuget-deps-tree
```

After we've installed the package, we can run the script from the repository. To run the unsigned script, you might need to open the PowerShell in Bypass execution policy mode (you can also set the policy permanently, but that is not advisable):

``` powershell
powershell.exe -executionpolicy Bypass
```

To run the script, use the following command:

``` powershell
./nailDownNuget.ps1 <path-to-sln> <package-name> <package-version>
```

This will print all dependency paths which include the specified package and version. Using that information we can determine which dependencies use the vulnerable one:

1. Try to update the direct dependency to the newest possible version.
2. Run the vulnerability check command again to see if the update fixed the issue
    1. If not, add the transitive dependency as a direct dependency of the project (just check beforehand that the version you're adding has fixed the vulnerability) and then repeat the previous step.
    2. If yes, that's it, we're done!

## Vulnerability checks in build pipelines

The vulnerability check command has the option to print the results in JSON format. That makes it easy to use in an automated way, so why don't we add it to our build pipelines?

To do that, we must determine what to do with the results we get. Do we stop the pipeline if we find a vulnerability? If so, what severity levels are acceptable? Also, how do we determine whether the issue is afflicting our system? As discussed in the sections above, these answers are hard to answer without starting with "Well, it depends".

As important as having high security standards is, we don't want to block the development process with false positives. Blocking the pipeline for each found vulnerability would do exactly that, which is why we want to avoid this solution. But that doesn't mean that the process cannot be automated in any way.

To get around the blocking issue, we've opted to go with a different solution by creating a separate scheduled pipeline that will run the command and notify the developers about the found vulnerabilities via Slack or email. That way we can always be on top of those issues without disturbing the development process. Depending on our needs, we can also add some filtering options (e.g. skip scanning the projects we're not deploying anywhere) and package white-listing (keeping track of vulnerable packages that do not afflict our system).