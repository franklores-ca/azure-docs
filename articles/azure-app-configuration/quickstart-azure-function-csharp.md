---
title: Quickstart for Azure App Configuration with Azure Functions | Microsoft Docs
description: A quickstart for using Azure App Configuration with Azure Functions.
services: azure-app-configuration
documentationcenter: ''
author: yegu-ms
manager: balans
editor: ''

ms.assetid: 
ms.service: azure-app-configuration
ms.devlang: csharp
ms.topic: quickstart
ms.tgt_pltfrm: Azure Functions
ms.workload: tbd
ms.date: 02/24/2019
ms.author: yegu

#Customer intent: As an Azure function developer, I want to manage all my app settings in one place.
---
# Quickstart: Create an Azure function with App Configuration

Azure App Configuration is a managed configuration service in Azure. It lets you easily store and manage all your application settings in one place that is separated from your code. This quickstart shows you how to incorporate the service into an Azure Function. 

You can use any code editor to complete the steps in this quickstart. However, [Visual Studio Code](https://code.visualstudio.com/) is an excellent option available on the Windows, macOS, and Linux platforms.

![Quickstart Complete local](./media/quickstarts/dotnet-core-function-launch-local.png)

## Prerequisites

To complete this quickstart, install [Visual Studio 2017](https://visualstudio.microsoft.com/vs) (and ensure that the **Azure development** workload is also installed) and the [latest Azure Functions tools](../azure-functions/functions-develop-vs.md#check-your-tools-version).

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## Create an app configuration store

[!INCLUDE [azure-app-configuration-create](../../includes/azure-app-configuration-create.md)]

## Create a function app

[!INCLUDE [Create a project using the Azure Functions template](../../includes/functions-vstools-create.md)]

## Connect to app configuration store

1. Open *Function1.cs* and add a reference to App Configuration .NET Core configuration provider.

    ```csharp
    using Microsoft.Extensions.Configuration.AzureAppConfiguration;
    ```

2. Update the `Run` method to use App Configuration by calling `builder.AddAzureAppConfiguration()`.

    ```csharp
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req, ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        var builder = new ConfigurationBuilder();
        builder.AddAzureAppConfiguration(Environment.GetEnvironmentVariable("ConnectionString"));
        var config = builder.Build();
        string message = config["TestApp:Settings:Message"];
        message = message ?? req.Query["message"];

        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        message = message ?? data?.message;

        return message != null
            ? (ActionResult) new OkObjectResult(message)
            : new BadRequestObjectResult("Please pass a message from a configuration store, on the query string or in the request body");
    }
    ```

## Test the function locally

1. Set an environment variable named **ConnectionString** and set it to the access key to your app configuration store. If you are using Windows Command Prompt, execute the following command and restart the Command Prompt to allow the change to take effect:

        setx ConnectionString "Endpoint=<service_endpoint>;Id=<store_id>;Secret=<secret_key>="

    If you are using Windows PowerShell, execute the following command:

        $Env:ConnectionString = "Endpoint=<service_endpoint>;Id=<store_id>;Secret=<secret_key>="

    If you are using macOS or Linux, execute the following command:

        export ConnectionString='Endpoint=<service_endpoint>;Id=<store_id>;Secret=<secret_key>='

2. To test your function, press **F5**. If prompted, accept the request from Visual Studio to download and install **Azure Functions Core (CLI)** tools. You may also need to enable a firewall exception so that the tools can handle HTTP requests.

3. Copy the URL of your function from the Azure Functions runtime output.

    ![Quickstart Function debugging in VS](./media/quickstarts/function-visual-studio-debugging.png)

4. Paste the URL for the HTTP request into your browser's address bar. The following shows the response in the browser to the local GET request returned by the function:

    ![Quickstart Function launch local](./media/quickstarts/dotnet-core-function-launch-local.png)

## Clean up resources

[!INCLUDE [azure-app-configuration-cleanup](../../includes/azure-app-configuration-cleanup.md)]

## Next steps

In this quickstart, you've created a new app configuration store and used it with an Azure function. To learn more about using App Configuration, continue to the next tutorial that demonstrates authentication.

> [!div class="nextstepaction"]
> [Managed Identities for Azure Resources Integration](./integrate-azure-managed-service-identity.md)
