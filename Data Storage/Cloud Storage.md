Cloud services are providing new options when choosing storage for your applications. While there are several provider options available, we will mainly focus on Azure Storage in this handbook.

## Azure Storage

Azure Storage is Microsoft's solution for cloud storages. There are a couple of storage options within the Azure ecosystem, which will be discussed in this section
[Official documentation](https://docs.microsoft.com/en-us/azure/storage/common/storage-configure-connection-string)

For local development, Microsoft has provided [Azurite](https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azurite?tabs=visual-studio) (previously called [Azure Storage Emulator](https://docs.microsoft.com/en-us/azure/storage/common/storage-use-emulator)). Azurite is an emulator that provides all the features available on Azure Storage: Blob Storage, Queues and Table storages. The fastest way to connect to your local Azure storage emulator is by setting the connection string to `UseDevelopmentStorage=true`, which is equivalent to:

```c#
DefaultEndpointsProtocol=http;
AccountName=devstoreaccount1;
AccountKey=Eby8vdM02xNOcqFlqUwJPLlmEtlCDXJ1OUzFT50uSRZ6IFsuFq2UVErCz4I6tq/K1SZFPTOtr/KBHBeksoGMGw==;
BlobEndpoint=http://127.0.0.1:10000/devstoreaccount1;
QueueEndpoint=http://127.0.0.1:10001/devstoreaccount1;
TableEndpoint=http://127.0.0.1:10002/devstoreaccount1;
```

This connection string can be used to connect to Blob, Queue and Table storages. If you want to see what data you added to your local storage, we recommend using [Azure Storage Explorer](https://azure.microsoft.com/en-us/features/storage-explorer/#features). This tool can also be used to connect to other storage accounts.

### Azure Blob Storage

Azure Blob storage is optimized for storing massive amounts of unstructured data like images, documents and other files. You can read more about it [here](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blobs-introduction).

Blob storage offers three types of resources:

- **Storage account** - a unique namespace in Azure for your data. Every object stored in Azure Storage has an address that consists of the account name and the Blob Storage endpoint, e.g. if your storage account is named `myaccount`, then the default endpoint for the storage is `http://myaccount.blob.core.windows.net`
- **Containers** - organizes a set of blobs, similar to a directory in a file system. A storage account can include an unlimited number of containers, and a container can store an unlimited number of blobs.
- **Blobs** - blobs store the actual data. There are three types of blobs:
  - Block blobs - store text and binary data, made up of blocks of data that can be managed individually, can store up to about 190.7 TiB
  - Append blobs - made up of blocks like block blobs, but are optimized for append operations, ideal for scenarios such as logging data from virtual machines.
  - Page blobs - store random access files up to 8 TiB in size, they store virtual hard drive files and serve as disks for Azure virtual machines

#### Access control

Since we use Azure Blob to store images and documents, it is important to manage blobs and containers in a way that allows only authorized users and services to access those files. We can do that in a couple of ways:

- Access Blob storage only through our API - that way we can enforce any authorization and validation rules we want before allowing a user to upload/download data from the storage, but that also means that all that data is flowing through our API, which can be costly because it is using up both a lot of bandwidth as well as additional computing power.
- Shared access signatures (SAS) - Azure Blob Storage supports generating SAS tokens that allow access to a specific blob or container for a specified period. By combining the SAS token and resource URIs, we can generate a URI and distribute it to client applications which can then access the blob directly, which reduces the bandwidth used to manage the files.
- Azure Active Directory (AD) - Azure Storage supports using Azure AD to authorize requests to blob data. We can use Azure role-based access control to grant permissions to a security principal, which may be a user, group, or application service principal.

#### Usage example

The main entry point for communication with Azure Blob Storage is `BlobServiceClient`. There are a couple of ways to create the client, we can use the extension method for `IServiceCollection` that registers the client. The extension method will register the client in the DI service:

```c#
// in Program.cs:
builder.Services.AddAzureClients(
  builder => builder.AddBlobServiceClient(builder.Configuration["BlobConfiguration:ConnectionString"]));

...

public class BlobService
{
  private readonly BlobServiceClient _blobServiceClient;

  public BlobService(BlobServiceClient blobServiceClient)
  {
    _blobServiceClient = blobServiceClient;
  }

  ...
}

```

Alternatively, we could build our own factory that creates the client:

```c#
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

### Azure Table Storage

[Azure Table Storage](https://docs.microsoft.com/en-us/azure/storage/tables/) provides a way to store large amounts of structured data. This service is a NoSQL database. We must note that this is not a replacement for an SQL database. For more information, please see [Understanding the differences between NoSQL and Relational Databases](https://docs.microsoft.com/en-us/azure/cosmos-db/relational-nosql).

Use it when you want to:

- Store data that doesn't require complex joins, foreign keys or any relationship.
- Store data that is de-normalized.
- Have fast queries using a clustered index.
