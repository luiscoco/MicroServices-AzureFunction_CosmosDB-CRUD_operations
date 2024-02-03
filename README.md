# Azure Functions for CRUD operations with Azure CosmosDB

## 1. Prerequisite

Create Azure CosmosDB account, database and collection

e navigate to **Azure CosmosDB** service and we click in this option to create a new account:

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB/assets/32194879/3822db85-173d-411d-8383-7f97d07c05f4)

We click on **Azure CosmosDB account** button

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB/assets/32194879/c1080788-bfc0-445e-9084-ce0f0772044a)

Now we select the option **Azure Cosmos DB for NoSQL**, and we press the **create** button

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB/assets/32194879/69182e3f-9493-41a8-ab6b-92308a9bdddf)

In following screen we input the required data for creating the service

We create a new **ResourceGroup name**: myRG

We set the **account name**: mycosmosdbluis1974

We choose the service **location**: France Central

Capacity mode: **serverless**

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB/assets/32194879/ed5a1407-0303-44b3-b528-7a7fc1398d80)

We navigate to the **Data Explorer** page and we create a **New Database** and a **New Container**

We first create a **New Database**. We input the **DatabaseId**: ToDoList

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB/assets/32194879/978b2f67-01c3-4711-95fd-e0eeb75a99c1)

We also create a **New Container**: Items

![image](https://github.com/luiscoco/MicroServices_dotNET8_CRUD_WebAPI-AzureCosmosDB/assets/32194879/10e1d766-9c53-4ab0-9624-52f257340480)

![image](https://github.com/luiscoco/MicroServices-AzureFunction_CosmosDB-CRUD_operations/assets/32194879/a7f40ff9-af30-4d1d-b04a-55fe66d7b4bb)

We insert the new items in the Azure CosmosDB

This is the **new item** json file:

```json
{
  "id": "1",
  "name": "Sample Item",
  "todoItem": "Sample Category"
}
```

We click in **Items** and then **New Item**

Then we copy and paste the json content and press **Save** button

![image](https://github.com/luiscoco/MicroServices-AzureFunction_CosmosDB-CRUD_operations/assets/32194879/08809426-85e4-4e75-a938-6d779989b218)

## 2. Create Azure Functions Project

We install the Project Templates with dotnet

```
dotnet new --install Microsoft.Azure.Functions.Worker.ProjectTemplates
```

First, we create a new Azure Functions project targeting .NET 8

```
dotnet new func -n CosmosDbCrudFunctions --Framework net8.0
cd CosmosDbCrudFunctions
```

![image](https://github.com/luiscoco/MicroServices-AzureFunction_CosmosDB-CRUD_operations/assets/32194879/45cb80bc-d2ce-497e-82b0-2e4dc0f587db)

## 3. Add NuGet Package for Cosmos DB

We add the necessary NuGet package **Microsoft.Azure.WebJobs.Extensions.CosmosDB**

```
dotnet add package Microsoft.Azure.WebJobs.Extensions.CosmosDB
```

![image](https://github.com/luiscoco/MicroServices-AzureFunction_CosmosDB-CRUD_operations/assets/32194879/37b60fa9-f8ef-438f-903a-807e29406a6c)

We also add the package **Microsoft.Azure.WebJobs.Extensions.Http**

![image](https://github.com/luiscoco/MicroServices-AzureFunction_CosmosDB-CRUD_operations/assets/32194879/f308274c-378e-420c-bd1b-db24e485460b)

## 4. Implement CRUD Operations

### 4.1. Create a new Item

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

### 4.2. Read Operation: Retrieve an Item by ID

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

### 4.3. Update Operation: Update an Item by ID

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

### 4.4. Delete Operation: Delete an Item by ID

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

## 5. Configure Local Settings

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

## 6. Run and test the Azure Function with Postman


