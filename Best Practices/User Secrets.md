In Visual Studio, by right-clicking on a project, you can select **Manage User Secrets**. It provides an alternative to the **appSettings.json** file that overrides its values. Use this to store any type of credentials or sensitive information that is related to your local setup.

User secrets are stored on your local machine and are ignored by Git. Sections that are not provided in user secrets will be taken from **appSettings.json**.

It is strongly advised against pushing any kind of usernames or passwords to Git if possible.

Note that user secrets are not a replacement for **appSettings.json** and they should contain all keys required for the application to function. Default values for other developers or staging machines should be stored in **appSettings.json**.

Sample of **appSettings.json**:

```json
{
	"ConnectionStrings":{
		"PrimaryDb": "Server=localhost;Port5432;Database:PrimaryDb;User Id=;Password=;"
	},
    "AdditionalSettings":{
        "AuthenticationUrl":"https://this.is.not.sensitive.info.com"
    }
}
```

Sample of **user secrets**:

```json
{
	"ConnectionStrings":{
		"PrimaryDb": "Server=localhost;Port5432;Database:LocalTestDb;User Id=myUsername;Password=myPassword;"
	}
}
```
