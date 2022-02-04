Cloud provided new options when choosing storage for your applications. Although there are several provider options, we will focus mainly on Azure Storage in this handbook.


### Azure Storage

[Official documentation](https://docs.microsoft.com/en-us/azure/storage/common/storage-configure-connection-string)

For development we use tool called [Azure storage emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator). Fastest way to connect to your local Azure storage is by setting `UseDevelopmentStorage=true`.

This is equivalent of:

```c#
DefaultEndpointsProtocol=http;
AccountName=devstoreaccount1;
AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;
TableEndpoint=http://127.0.0.1:10001/devstoreaccount1;
```

And you can use it to connect to Blob, Queue and Table storages. If you want to see what data you added to your local storage, you can use [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/#features).



#### Azure Blob Storage

Azure Blob storage is optimized for storing massive amounts of unstructured data like images, documents and other files. You can read more about it [here](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction).

Blob storage offers three types of resources:

#### Storage account:

- A unique namespace in Azure for your data
- If your storage account is named `myaccount`, then the default endpoint for blob is: `http://myaccount.blob.core.windows.net`

#### Containers:

- Similar to folders (we have files in a folder)
- Organizes a set of blobs (we have blobs in a container)
- You can have multiple containers (eg. photos, documents, etc.)

#### Blobs:

- Block blobs
- Append blobs
- Page blobs



```c#
// Configuration example

public interface IAzureBlobServiceClientFactory
{
    BlobServiceClient CreateBlobServiceClient();
}

public class AzureBlobOptions
{
    public string ConnectionString { get; set; }
}

public class AzureBlobServiceClientFactory : IAzureBlobServiceClientFactory
{
    private readonly IOptions<AzureBlobOptions> _options;
    
    public AzureBlobServiceClientFactory(IOptions<AzureBlobOptions> options)
    {
        _options = options;
    }
    
    public BlobServiceClient CreateBlobServiceClient()
    {
        return new BlobServiceClient(_options.Value.ConnectionString);
    }
}
```

```c#
// Usage example

public class TestService
{
    private readonly BlobServiceClient _client;

    public TestService(IAzureBlobServiceClientFactory azureBlobServiceClientFactory)
    {
        _client = azureBlobServiceClientFactory.CreateBlobServiceClient();
    }

    public async Task CreateContainerAsync(string containerName)
    {
        await _client.CreateContainerAsync(containerName);
    }
}
```

&nbsp;

#### Azure Table Storage

Azure Table Storage provides a way to store large amounts of structured data. This service is a NoSQL database. You can read more about it [here](https://docs.microsoft.com/en-us/azure/storage/tables/).

We must note that this is not a replacement for SQL database. For more information, please see [Understanding the differences between NoSQL and Relational Databases.](https://docs.microsoft.com/en-us/azure/cosmos-db/relational-nosql)

Use it when you want to:
- Store data that doesn't require complex joins, foreign keys or any relationship.
- Store data which is de-normalized.
- Have fast queries using a clustered index.
