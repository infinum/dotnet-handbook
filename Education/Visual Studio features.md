For developing purposes, most of us are using Visual Studio (VS) as our main IDE. VS is built with many features and capabilities for various programming languages but we will share some useful tips for .NET development.



### Project templates

VS comes with a list of predefined templates that you can choose from when starting a new project. It is important to select the right project template as it can generate code or sections of code that allow you to focus on implementing business requirements instead of messing with a development infrastructure.

Some of the templates that are most commonly used:

- Console App - contains predefined Program.cs class that can be run as a console application.
- Class Library - a base project template for class libraries.
- ASP.NET Core Web API - contains a structure required for a project to be run as API with a mock controller.
- xUnit Test Project - a base project for xUnit tests.
- Azure Functions - contains a structure for running a project as an Azure function.

Some of these templates provide an additional configuration in the setup itself so the code generated is immediately ready to use. For example, the Azure Functions project allows you to choose what type of trigger you want for your function. It also allows you to setup a connection string to the trigger so the generated code can be run and debugged locally.

Visual Studio also allows you to create your own templates for the project. The instructions on how to create your custom template can be found on this [link](https://docs.microsoft.com/en-us/visualstudio/ide/how-to-create-project-templates?view=vs-2022).



### Extensions

VS allows you to add extensions that enhance or alter functionalities provided by the IDE. There are a lot of extensions that can be found at the 'Extensions' tab or searched on the internet. Some of the extensions are free and some of them are paid. 

Some free extensions that we suggest:

- Fine Code Coverage - provides an overview of your test coverage giving information such as the percentage of code covered, risky hotspot, covered branches etc.
- Format document on Save - automatically applies default code formatting when saving a document.
- Output enhancer - provides better visual graphics when checking IDE console logs.
- Visual Studio Spell Checker - adds spell-checking for all files.



### App settings configuration

Different projects require different setting configurations, like connection string to database or URL to external API etc. These configurations in .NET are usually stored in the appsettings.json file. Visual Studio allows you to create different configurations through 'Configuration Manager' found in the 'Build' tab. By default, projects are run in the 'Debug' configuration with appsettings.json but it can be changed to use different settings, for example, appsettings.Test.json.

Additionally VS allows you to use 'user secrets' by right-clicking on the project and clicking on 'Manage User Secrets'. These secrets are stored on your machine and cannot be pushed to GitHub. These secrets override appsettings.json and are useful for local development.



### Connecting to Azure 

VS can be connected to an Azure account by clicking on 'Manage connections' in the 'Team' tab. When VS is connected to Azure it allows you to use several different features, such as accessing the DevOps git repository, selecting specific existing trigger (like blob or queue) from your Azure account etc. These Azure connections can also be used to connect to the Azure database or other resources and manage them through VS itself.



### Shortcuts

VS provides shortcuts to some most common actions.  All shortcuts can be found by clicking on 'Options' under the 'Tools' tab in the 'Keyboard' menu. Note that some extensions modify or add new shortcuts. Shortcuts can also be exported or imported.

Some of the most common that we use:

- `ctrl+s` - to save a file.
- `ctrl+e,d` - to format a file.
- `ctrl+w` - to close a file (needs to be mapped in the 'Keyboard' menu to command 'File.Close').
- `ctrl+t` - to navigate through files, properties, methods etc.
- `ctrl+g` - to navigate to a specific line number in the file. 
- `ctrl+f` - to search through files.
- `ctrl+h` - to open the find and replace menu. 
- `ctrl+q` - to search through the whole project.
- `alt+enter` or `ctrl+.`- to open InteliSense for the line you are at.
- `F12` - to navigate to the code definition.
- `ctrl+F12` - to navigate to method implementation or class definition.
- `ctrl+r,a` - to run all tests.
- `ctrl+d` - to make a copy of the line you are at.
- `F2` - to rename a file, property or method.
- `ctrl+shift+l` - to delete the current line.
- `shift+del` - to cut the current line.
- `ctrl+k+c` - to comment selected lines of code.
- `ctrl+k+u` - to uncomment selected lines of code.
- `ctrl+m+o` - to collapse all methods in the current file.
- `ctrl+m+p` - to expand all methods in the current file.



A list of helpful tips and tricks can be found on this [link](https://www.youtube.com/playlist?list=PLReL099Y5nRc-zbaFbf0aNcIamBQujOxP).