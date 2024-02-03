# Azure Function to Create an Item in Azure CosmosDB

## 1. Create Azure Functions Project

First, we create a new Azure Functions project targeting .NET 8

```
dotnet new func -n CosmosDbCrudFunctions --framework net8.0
cd CosmosDbCrudFunctions
```

## 2. Add NuGet Package for Cosmos DB

We add the necessary NuGet package for Cosmos DB integration

```
dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB --version <latest_version>
```

Replace <latest_version> with the latest version available at the time



## 3. Implement CRUD Operations

### 3.1. Create a new Item

You will need to add a new class for each operation or implement them in a single class based on your preference

Here's an example of how to implement a simple Create operation

Similar patterns can be followed for Read, Update, and Delete operations.

**CreateFunction.cs**

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System.IO;
using System.Threading.Tasks;

public static class CreateFunction
{
    [FunctionName("CreateItem")]
    public static async Task<IActionResult> CreateItem(
        [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
        [CosmosDB(
            databaseName: "YourDatabaseName",
            collectionName: "YourCollectionName",
            ConnectionStringSetting = "CosmosDBConnection")] IAsyncCollector<dynamic> items,
        ILogger log)
    {
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        await items.AddAsync(data);

        return new OkObjectResult(data);
    }
}
```

### 3.2. Read Operation: Retrieve an Item by ID

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using System.Threading.Tasks;

public static class ReadFunction
{
    [FunctionName("ReadItem")]
    public static async Task<IActionResult> ReadItem(
        [HttpTrigger(AuthorizationLevel.Function, "get", Route = "items/{id}")] HttpRequest req,
        [CosmosDB(
            databaseName: "YourDatabaseName",
            collectionName: "YourCollectionName",
            ConnectionStringSetting = "CosmosDBConnection",
            Id = "{id}",
            PartitionKey = "{id}")] dynamic item,
        ILogger log, string id)
    {
        if (item == null)
        {
            log.LogInformation($"Item not found for id: {id}");
            return new NotFoundResult();
        }
        return new OkObjectResult(item);
    }
}
```

### 3.3. Update Operation: Update an Item by ID

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using Newtonsoft.Json;
using System.IO;
using System.Threading.Tasks;

public static class UpdateFunction
{
    [FunctionName("UpdateItem")]
    public static async Task<IActionResult> UpdateItem(
        [HttpTrigger(AuthorizationLevel.Function, "put", Route = "items/{id}")] HttpRequest req,
        [CosmosDB(
            databaseName: "YourDatabaseName",
            collectionName: "YourCollectionName",
            ConnectionStringSetting = "CosmosDBConnection",
            Id = "{id}",
            PartitionKey = "{id}")] IAsyncCollector<dynamic> itemsOut,
        ILogger log, string id)
    {
        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        data.id = id; // Ensure the ID is set correctly for the item to be updated

        await itemsOut.AddAsync(data);

        return new OkObjectResult(data);
    }
}
```

### 3.4. Delete Operation: Delete an Item by ID

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using System.Threading.Tasks;

public static class DeleteFunction
{
    [FunctionName("DeleteItem")]
    public static async Task<IActionResult> DeleteItem(
        [HttpTrigger(AuthorizationLevel.Function, "delete", Route = "items/{id}")] HttpRequest req,
        [CosmosDB(
            databaseName: "YourDatabaseName",
            collectionName: "YourCollectionName",
            ConnectionStringSetting = "CosmosDBConnection",
            Id = "{id}",
            PartitionKey = "{id}")] dynamic item,
        [CosmosDB(
            databaseName: "YourDatabaseName",
            collectionName: "YourCollectionName",
            ConnectionStringSetting = "CosmosDBConnection")] DocumentClient client,
        ILogger log, string id)
    {
        if (item == null)
        {
            log.LogInformation($"Item not found for deletion with id: {id}");
            return new NotFoundResult();
        }

        Uri documentUri = UriFactory.CreateDocumentUri("YourDatabaseName", "YourCollectionName", id);
        await client.DeleteDocumentAsync(documentUri, new RequestOptions { PartitionKey = new PartitionKey(id) });

        return new OkResult();
    }
}
```





## 4. Configure Local Settings

In your **local.settings.json** file, add your Cosmos DB connection string as follows

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "CosmosDBConnection": "Your_Cosmos_DB_Connection_String"
  }
}
```


