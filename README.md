# Azure chatbots

## How to create a basic bot for practice purposes

#### To start learning about the development of an Azure chatbot I recommend following [these](https://docs.microsoft.com/en-us/composer/quickstart-create-bot) instructions

The best option to start is a C# empty bot as the tutorial suggests

## How to provision resources for a multitenant chatbot

#### 1. Login to Azure:

   ```
   az login
   ```

   Save the following values:

   - `tenantId`
   - `id`

#### 2. To select a subscription, run:

   Replace:

   - < subscription >: `name` or `id` from step 1

   ```
   az account set --subscription "<subscription>"
   ```

#### 3. If you don't already have an appropriate resource group, run:

   Replace:

   - < group >: Choose a new name for your group
   - < region >: Choose a region having in consideration [this info](https://docs.microsoft.com/en-us/azure/cognitive-services/luis/luis-reference-regions) when using LUIS and the [failover](https://docs.microsoft.com/en-us/azure/availability-zones/cross-region-replication-azure) region also

   ```
   az group create --name "<group>" --location "<region>"
   ```

   Save the following values:

   - `name`
   - `region`

#### 4. Create a new identity resource

   Replace:

   - < app-registration-display-name >: Choose a new name for your identity resource

   ```
   az ad app create --display-name "<app-registration-display-name>" --sign-in-audience "AzureADandPersonalMicrosoftAccount"
   ```

   Save the following values:

   - `appId`
   - `name`

#### 5. Reset AppPassword

   Replace:

   - < appId >: `appId` from step 4

   ```
   az ad app credential reset --id "<appId>"
   ```

   Save the following values:

   - `password`

   > It's really important to store this value because you're not going to be able to see it again

#### 6. Create a Web App and App Service Plan resources

   Considerations:

   - At this point you still don't have an App Service Plan
   - For the command to be executed you must be in the root folder of this repository (where this README.md is located)

   For this step use the following template and parameters file:

   - Template: ./ARMTemplates/template-WebApp.json
   - Parameters: ./ARMTemplates/parameters-WebApp.json

   In "parameter-WebApp.json" replace:

   - < appServiceName >: A unique name for your WebApp
   - < newAppServicePlanName >: A unique name for your App Service Plan
   - < newAppServicePlanLocation >: `region` from step 3
   - < appId >: `appId` from step 4
   - < appSecret >: `password` from step 5

   Replace:

   - < resource-group >: `name` from step 3

   ```
   az deployment group create –-resource-group <resource-group> --template-file ./ARMTemplates/template-WebApp.json --parameters @./ARMTemplates/parameters-WebApp.json
   ```

   Save the following values:

   - `appServiceName`

#### 7. Create an Azure Bot resource

   Considerations:

   - For the command to be executed you must be in the root folder of this repository (where this README.md is located)

   For this step use the following template and parameters file:

   - Template: ./ARMTemplates/template-AzureBot.json
   - Parameters: ./ARMTemplates/parameters-AzureBot.json

   In "parameter-AzureBot.json" replace:

   - < azureBotId >: A unique name for your AzureBot
   - < botEndpoint >: You have to replace `appServiceName` in the following string and use it: "https://<appServiceName>.azurewebsites.net/api/messages"
   - < appId >: `appId` from step 4

   Replace:

   - < resource-group >: `name` from step 3

   ```
   az deployment group create --resource-group <resource-group> --template-file ./ARMTemplates/template-AzureBot.json --parameters @./ARMTemplates/parameters-AzureBot.json
   ```

   Save the following values:

   - `azureBotId`

#### 8. Reset AppPassword

   Finally, you need to create the following deploying template to use in composer
   Replace:

   - < appId >: `appId` from step 4

   ```json
   // Fill up this template with the info you got during the provisioning process

   {
     "name": "<This is a registry app>",
     "environment": "composer",
     "tenantId": "<tenant id of your azure account>",
     "hostname": "<web application host name>",
     "runtimeIdentifier": "win-x64",
     "resourceGroup": "<name of your resource group>",
     "botName": "<name of your bot channel registration>",
     "subscriptionId": "<id of your subscription>",
     "region": "<region of your resource group>",
     "appServiceOperatingSystem": "windows",
     "scmHostDomain": "",
     "luisResource": "<name of your luis resource>",
     "settings": {
       "applicationInsights": {
         "InstrumentationKey": "<Instrumentation Key>",
         "connectionString": "<connection string>"
       },
       "cosmosDb": {
         "cosmosDBEndpoint": "<endpoint url>",
         "authKey": "<auth key>",
         "databaseId": "botstate-db",
         "containerId": "botstate-container"
       },
       "blobStorage": {
         "connectionString": "<connection string>",
         "container": "<container>"
       },
       "luis": {
         "authoringKey": "",
         "authoringEndpoint": "",
         "endpointKey": "",
         "endpoint": "",
         "region": "westus"
       },
       "qna": {
         "subscriptionKey": "<subscription key>",
         "endpoint": "<endpoint>"
       },
       "MicrosoftAppId": "<app id from Bot Framework registration>",
       "MicrosoftAppPassword": "<app password from Bot Framework registration>"
     }
   }

    // This is an example of a filled template and the places where u can find the values to fill the blanks
    {
    "name": "wr88-ComposerTest", // Don't know what it does
    "environment": "composer", // "composer" when using Bot Framework Composer
    "tenantId": "b9718da2-3487-4208-a707-7eb06ff6173f", // Azure Active Directory -> Properties -> Tenant ID
    "hostname": "wr88-ComposerTest", // App Service -> Overview
    "runtimeIdentifier": "win-x64", // Default value
    "resourceGroup": "Bot-deployment-group", // Resource group -> Name (as is)
    "botName": "wr88-ComposerTest", // Azure Bot -> name (as is)
    "subscriptionId": "16799aee-6093-4464-b21a-230d6fa58125", // Subscriptions
    "region": "eastus", // Resource group -> Overview -> Location
    "appServiceOperatingSystem": "windows", // Required for C# bots
    "scmHostDomain": "", // Leave it as is
    "luisResource": "<name of your luis resource>", // [Optional] (Language understanding resource) -> Name (as is)
    "settings": {
    "applicationInsights": {
      "InstrumentationKey": "<Instrumentation Key>",
      "connectionString": "<connection string>"
    },
    "cosmosDb": {
      "cosmosDBEndpoint": "<endpoint url>",
      "authKey": "<auth key>",
      "databaseId": "botstate-db",
      "containerId": "botstate-container"
    },
    "blobStorage": {
      "connectionString": "<connection string>",
      "container": "<container>"
    },
    "luis": {
      "authoringKey": "", // LUIS resource authoring -> Key and Endpoint -> KEY 1
      "authoringEndpoint": "", // LUIS resource authoring -> Overview | Key and Endpoint -> Endpoint
      "endpointKey": "", // LUIS resource -> Key and Endpoint -> KEY 1
      "endpoint": "", // LUIS resource -> Overview | Key and Endpoint -> Endpoint
      "region": "westus" // LUIS resource authoring -> Overview -> Location
    },
    "qna": {
      "subscriptionKey": "<subscription key>",
      "endpoint": "<endpoint>"
    },
    "MicrosoftAppId": "faf2a831-b4d5-41be-af8a-33ee069a02a1", // Azure Active Directory -> App registration -> 'Select your app from the list' -> Overview
    "MicrosoftAppPassword": "wtp8Q~xBRXqNl0dt40Ekf~iLCg-5vo1YWJnsDaMh" // If you forget your password, with 'MicrosoftAppId' as 'appId' run the command in step 5
    }
    }

   ```

#### 9. Deploy from Bot Framework Composer

   - In Publish -> Publishin Profile -> Add New, give a name for the publishing profile and select Azure as target, then continue.
   - Select "Import existing resources" and paste the template you created in the last step, then continue.
   
   > For a more detailed explanation you can take a look at the official documentation [here](https://docs.microsoft.com/en-us/composer/how-to-publish-bot?tabs=v2x#import-existing-azure-resources)
   
## Publish your bot in Azure Cloud

#### To wrap things up, you can follow [this](https://docs.microsoft.com/en-us/composer/how-to-publish-bot?tabs=v2x#publish-your-bot) section of the official documentation

## Deployment on Web Page

#### To have a fully working chatbot embedded in a web page you can follow [these](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0) instructions
> Put special attention to the security considerations about why is better to use [option 1](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0#option-1---exchange-your-secret-for-a-token-and-generate-the-embed) instead of [option 2](https://docs.microsoft.com/en-us/azure/bot-service/bot-service-channel-connect-webchat?view=azure-bot-service-4.0#option-2---embed-the-web-chat-control-in-your-website-using-the-secret)

## 🎉🎉 Congratulations 🎉🎉, now you have a fully working chatbot embedded in your webpage
