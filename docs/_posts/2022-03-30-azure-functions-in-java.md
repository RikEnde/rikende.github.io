---
layout: post
title:  "Azure Functions in java"
date:   2022-03-30 12:00:00 -0500
tags: [azure,java,cloud]
---

## Functions fundamentals

Azure functions are Microsoft's equivalent of Lambda's in AWS: a so called `serverless` development model. In Azure, a
`function` is defined as: `an event plus the code that is being triggered by the event`.

Functions reside in a so called `Function App`, which is the unit of deployment for functions. One function app can 
contain multiple functions. Function apps run on top of A zure App Service, which is the platform as a service (PaaS) 
solution in Azure. This started with native support for functions written in .NET and PowerShell, but also support 
Node, Python, Java and 'Custom', which should include native binaries or docker, but these were outside the scope of 
this document.

Every function must have exactly one trigger, but can have zero or more output bindings.

#### Advantages

The advantages of Functions are the same as for serverless in general: you have no involvement in deployment, starting, 
stopping, scaling, monitoring etc. of a function, this is all done by the cloud provider. There is no billing for a 
function merely existing, the billing is based on a function of the number of executions and the amount of resources 
used by each execution.

There are different pricing plans. In the `consumption pricing plan`, the first 1 million executions per month are free 
(as far as I can tell), with billing based on so called `gigabyte-seconds: RAM used x CPU time used`. You can set monthly 
quota on a Function App, with alerts on soft and hard limits, in order to prevent surprise bills.

#### Caveats

Because of the way functions work, and the way they are billed, they are not suitable for implementing long running 
batch jobs. There is a limit of 5 minutes for a single execution (I’m not sure how hard this limit is), after which 
a hang check timer may be triggered. Because they are billed per gigabyte-second, it would save cost to make the 
runtime short, and keep the RAM use in check. I would recommend streaming I/O where possible, rather than reading a 
huge chunk of data in memory, processing it in memory and then storing the result.



## Implementation

Implementation
A function resides in a Function App, which resides under a resource group. A blob store resides in a storage account, 
which resides under a resource group, so there is a number of resources we need to create and / or manage. As a first 
step, I recommend installing the `AzureCLI`

Mac:

```bash
brew install azure-cli
```

Windows:

<https://aka.ms/installazurecliwindows>

For working with azure functions, install the `Azure Functions Core Tools`.

Mac:

```bash
brew tap azure/functions
brew install azure-functions-core-tools@3
```

Windows: 

<https://go.microsoft.com/fwlink/?linkid=2135274>

#### Azure CLI

To log on to azure:

```bash
az login
```

This will open a browser window with a prompt to log on to your microsoft account. After doing this, the shell you 
issued this command from will be authenticated with azure.

#### Subscription

Show a list of subscription available to you:
```bash
az account list --output table
```

Select a subscription:
```bash
az account set -s "$subscriptionName"
```

You can see the list of available commands with az help.

#### Resource group

Create a resource group. Use the location name, not display name.
```bash
az account list-locations -o table
az group create --location "$location" --resource-group "$resourceGroup"
```

#### Storage account

A storage account can be created like this:
```bash
az storage account create -n "$storageAccount" \ 
    -l "$location" -g "$resourceGroup" \ 
    --sku "Standard_LRS"
```

For monitoring and performance monitoring we create an `application insights` resource. The selected storage account 
must be of account kind: `Account kind StorageV2 (general purpose v2)` in order for triggers to be able to work.

#### Storage containers

We can create storage containers with the following command:
```bash
az storage container create -n myblob \ 
    --account-name $"storageAccount" \ 
    --resource-group "$resourceGroup"\ 
    --fail-on-exist
```

This command creates a storage container called `myblob` under the selected storage account.

#### Function App

Now we can create the function app. Keep in mind that this is not the executable code, but a kind of container in 
azure for the actual functions that we will deploy later.

```bash
az functionapp create \
    -n "$applicationName" \
    -g "$resourceGroup" \
    --storage-account "$storageAccount" \ 
    --consumption-plan-location "$location" \ 
    --runtime "java"
```


#### Application insights

An application insights is a resource that can be used for monitoring, logging, testing etc of an application in Azure.

```bash
az monitor app-insights component create --app "$applicationName" \ 
    --location "$location" --kind web -g "$resourceGroup" \ 
    --application-type web
```

The application insights provides the following functionality to our function:
![Application insights](/images/2022-03-30/application_insights.png)

Now the infrastructure should be complete that we can deploy our functions to.


## Azure Functions Core Tools

A Functions project directory contains the files `host.json` and `local.settings.json`, along with subfolders that 
contain the code for individual functions.

Create a new function project:


```bash
func init
```

Create a new function in the function project:
```bash
func new
```

This command provides a menu with different runtimes to choose from. For java close custom. The next menu provides 
choices of different types of functions by the type of trigger.

For a java function you can create the project using a maven archetype:

```bash
mvn archetype:generate \ 
    -DarchetypeGroupId=com.microsoft.azure \ 
    -DarchetypeArtifactId=azure-functions-archetype
```

## Azure functions maven plugin

All roads to deploying java functions to Azure, whether it is done from a developer laptop, `github actions` or 
`Jenkins`, start with the Microsoft `azure-functions-maven-plugin`. A deployable artifact is a zip file with the 
following structure:
```bash
├── azure-function-0.0.1-SNAPSHOT.jar
├── blobProcessor
│ └── function.json ├── host.json
└── local.settings.json
```

The Microsoft `azure-functions-maven-plugin` creates this for you:

```bash
mvn package azure-functions:package
cd target/azure-functions/azure-function-20210628/ 
zip -r azure-function-20210628 .
```

This creates the deployable zipfile that can be deployed with this command:

```bash
az functionapp deploy --resource-group feiseu2-supply-rg-001 \ 
    --name azure-function-20210628 \
    --src-path ./azure-function-20210628.zip \
    --type jar --async true \
    --ignore-stack true
```

These are a lot of moving parts, that the maven plugin can do automatically with the goal:

```bash
mvn azure-functions:deploy
```

Which takes its settings from the pom.xml

```xml
<dependency>
    <groupId>com.microsoft.azure.functions</groupId>
    <artifactId>azure-functions-java-library</artifactId>
    <version>1.4.2</version>
</dependency> 

<!-- (...) -->

<plugin>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>azure-functions-maven-plugin</artifactId>
    <version>${azure.functions.maven.plugin.version}</version>
    <configuration>
        <appName>${functionAppName}</appName>
        <region>${functionAppRegion}</region>
        <resourceGroup>${resourceGroup}</resourceGroup>
        <appServicePlanName>${appServicePlan}</appServicePlanName>
        <subscriptionId>${subscriptionId}</subscriptionId>
        <runtime>
            <!-- runtime os, could be windows, linux or docker-->
            <os>windows</os> 
            <javaVersion>11</javaVersion>
        </runtime>
        <appSettings>
            <property>
                <name>FUNCTIONS_EXTENSION_VERSION</name>
                <value>~3</value>
            </property>
        </appSettings>
    </configuration>
    <executions>
        <execution>
            <id>package-functions</id>
            <goals>
                <goal>package</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

## Show me the code


And now for the Java code. Here we define the function. A function corresponds with a method annotated with the 
`@FunctionName` annotation. The next annotation, `@BlobOutput`, defines where the output of the function, in this case 
the return value of the method, will go: a blob store named `spring-poc/{name}-Copy`. The `{name}` part is defined 
later. The datatype in this case is specified as `binary`. The other options are `stream` and `string`. The connection 
`AzureWebJobsStorage` is a key you will find on the configuration tab of the Function App in the Azure UI, or in the 
configuration settings from the CLI:


```bash
az functionapp config appsettings list \
    -g "$resourceGroup" -n "$applicationName" -o table
```

The connection string it contains is the one that belongs to the storage account that the storage container we are 
accessing is in. You can find these in the Azure UI under access keys, or from the CLI with the following command:

```bash
az storage account show-connection-string \
    --name "$storageAccount" --resource-group "$resourceGroup"
```

One of the method parameters is annotated as a `@BlobTrigger`. The event that will trigger this function is a change 
to the blob store we reference here, that matches the path `blobby/{name}`. The `{name}` path parameter is bound to the 
filename parameter with the `@BindingName` annotation. The execution context enables limited interaction with the 
function's execution environment, for example for logging. If you have an application insights defined for this 
function app, you have more fine grained interactions with the execution environment available via te UI, where you 
can even open a command line in the container the function is running in.

```java
public class BlobFunction {
    @FunctionName("blobProcessor") 
    @BlobOutput(name = "target", path = "spring-poc/{name}-Copy", connection = "AzureWebJobsStorage", dataType = "binary")
    public byte[] run( @BlobTrigger(name = "file", 
                       dataType = "binary", 
                       path = "blobby/{name}", 
                       connection = "AzureWebJobsStorage") byte[] content,
                       @BindingName("name") String filename, 
        final ExecutionContext context) {
            if (content != null) {
                context.getLogger().info("A file named \"" + filename + "\" arrived of size: " + content.length); 
            } else {
                context.getLogger().log(Level.SEVERE, "A file content null with name " + filename);
            }
            return content;
        } 
}
```

