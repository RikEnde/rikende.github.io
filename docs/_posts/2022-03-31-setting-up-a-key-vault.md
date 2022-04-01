---
layout: post
title:  "Setting up a Key Vault in Azure"
date:   2022-03-31 12:00:00 -0500
tags: [azure,config,cloud]
---

## Setting up a Key Vault

Here is how to set up a key vault using the Azure CLI

Select subscription

```bash
az login
az account set -s "$subscriptionName"
```

### Create a Service Principal

To authenticate your application with key vault it needs a Service principal, which is an identity in Active Directory.
Here is how you can create one from the CLI:

```bash
az ad sp create-for-rbac --name key-vault-poc-sp-1
```

You can query the settings and ids for this spn with the following command:

```bash
az ad sp list --display-name "${service_principal_display_name}"
```

This will return a JSON representation of the service principal. Look for the propery `servicePrincipalNames` which 
contains a UUID. You will need this spn later.

```bash
"servicePrincipalNames": [ "aaaaaaaa-bbbb-cccc-dddd-012345678901" ],
```

### Create a key vault instance


```bash
az keyvault create \
  --resource-group "$resourceGroup" \ 
  --name "$keyVaultName" \ 
  --enabled-for-deployment true \ 
  --enabled-for-disk-encryption true \ 
  --enabled-for-template-deployment true \ 
  --location "$location" \
  --query properties.vaultUri \
  --sku standard
```

This should return the vault's Uri. For example: `https://key-vault-spring-poc-1.vault.azure.net/` 
When you create the key vault from the CLI, Azure will add access policies for your personal azure identity.

To add an access policy for the service principal made for us by cloud services:

```bash
az keyvault set-policy --name "$keyVaultName" --spn "${service_principal_names_uuid}" --secret-permissions get list
```

This will allow our application to list and read keys, nothing more. 

#### Set a value

```bash
az keyvault secret set --name "greeting" --vault-name "$keyVaultName" --value "Hello World"
```

#### Load application properties from Key Vault

For a spring application use the app_id from the service principal for client-id and the clientSecret for the 
client-key. The tenant-id can be found in the json properties of your application.

```bash
azure.keyvault.client-id=[client id] 
azure.keyvault.client-key=[client secret]
azure.keyvault.enabled=true
azure.keyvault.tenant-id=[tenant id] 
azure.keyvault.uri=https://key-vault-spring-poc-2.vault.azure.net/
```

Any additional properties stored in this key vault will be loaded as application properties. Keep in mind that 
Azure Key Vault does not support dots in key names. Spring will allow you to substitute those for dashes. This does 
mean that application properties with both dots and dashes in the name are not supported. You can substitute a dash by 
capitalizing the next character, for example `azure.keyvault.client-key -> azure-keyvault-clientKey`. There is no 
realistic scenario where you would store this particular key in Key Vault, I just couldn't think of another example 
with both dots and dashes off the top of my head. 

For example:

```bash
az keyvault secret set --name "spring-datasource-username" --vault-name "$keyVaultName" --value "username"
az keyvault secret set --name "spring-datasource-password" --vault-name "$keyVaultName" --value "password"
az keyvault secret set --name "spring-datasource-url" --vault-name "$keyVaultName" --value "jdbc:sqlserver://databaseserver.database.windows.net:1433;database=db_name;etc..."
```
