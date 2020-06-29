```
   _____                               
  /  _  \ __________ _________   ____  
 /  /_\  \\___   /  |  \_  __ \_/ __ \ 
/    |    \/    /|  |  /|  | \/\  ___/ 
\____|__  /_____ \____/ |__|    \___  >
        \/      \/                  \/ 
```
# Reference

- Azure Cli: https://docs.microsoft.com/cli/azure
- Azure Tools: https://azure.microsoft.com/downloads

# First Steps

## Login into Azure

First you need to login into the Azure account, with a Cloud Admin user because it's needed to create some Azure AD policies, a regular cloud user cannot create or apply any resource in the AD.

    > az login

This login will redirect you to the Portal, login with your required credentials after that the web will redirect you and a JSON message will be prompt in your terminal. From all the showed values, the "id" one is the Subscription ID needed to continue creating some other resources, so remember/copy it.

    Output:
    ```
    [
    {
        "cloudName": "AzureCloud",
        [...]
        "id": "12345678-1234-1234-1234-0123456789ab",
        [...]
        }
    }
    ]
    ```
If you're already logged in, you can run this other command to check your current Subscrition ID:

    > az account show

This command will show you the actual Subscription ID.

    Output:
    ```
    [
    {
        "cloudName": "AzureCloud",
        [...]
        "id": "12345678-1234-1234-1234-0123456789ab",
        [...]
        }
    }
    ]
    ```

Maybe your Azure account have different Subscriptions, there is a way to show them all and set the correct one:

    > az account list --query "[].{name:name, subscriptionId:id}"

This command will show all the previously logged Azure Subscrition ID accounts.

    Output:
    ```
    [
      {
        "name": "Pay-As-You-Go",
        "subscriptionId": "12345678-1234-1234-1234-0123456789ab"
      },
      {
        "name": "Subscrition A",
        "subscriptionId": "12345678-1234-1234-1234-0123456789cd"
      },
      {
        "name": "Subscrition B",
        "subscriptionId": "12345678-1234-1234-1234-0123456789ef"
      },
      {
        "name": "Subscrition C",
        "subscriptionId": "12345678-1234-1234-1234-0123456789gh"
      },
      [...]
    ]
    ```
Now, you can choose the one that will be used to bill all the Azure resources that will be deployed with Terraform, use this command to set the requiered account:

    > az account set --subscription="<subscription_id>"

There is no output with this command, so its reccomended to relaunch "show" command again:

    > az account show

## Create Azure credentials: Azure Service Principal

With the Subscription ID, we need to create a role in the Azure Portal that will allow us to create any resource on the subscription, we must run this command with that ID:

    > az ad sp create-for-rbac --role="Contributor" --scopes="/subscriptions/<subscription_id>" --name "terraform"

This command will create a SP credential with the provided name, in this case "terraform". Be yourself to generate any name this value is a string.

    Output:
    ```
    {
      "appId": "12345678-1234-1234-1234-0123456789ab",
      "displayName": "terraform",
      "name": "http://terraform",
      "password": "12345678-1234-1234-1234-0123456789ab",
      "tenant": "12345678-1234-1234-1234-0123456789ab"
    }
    ```
## Using your Azure SP credentials

There are 2 different ways to use these Azure SP credentials:

### Login with Azure SP credentials

    > az login --service-principal -u <appID> -p "<password>" --tenant "<tenant>"
Output:
```
[
  {
    "cloudName": "AzureCloud",
    [...]
    "id": "12345678-1234-1234-1234-0123456789ab",
    [...]
    }
  }
]
```
Now you're logged in with these Azure SP, this ones will persist in your computer almost forever. Not really recomended.

### Use Credentials with Terraform Provider

I higly recommend this other one, because it's more secure also you will be able to revoke it via Azure Cli any case:

Set your providers.tf file with the following parameters

```hcl
variable "client_secret" {
}

provider "azurerm" {

  version = "<3.0,>=2.4"

  subscription_id = "12345678-1234-1234-1234-0123456789ab"
  client_id       = "12345678-1234-1234-1234-0123456789ab"
  client_secret   = var.client_secret
  tenant_id       = "12345678-1234-1234-1234-0123456789ab"

  features {}
}
```

This will force you to run terraform these ways:

- Command line
    > terraform apply -var="client_secret=12345678-1234-1234-1234-0123456789ab"

- Exporting as a Environment Variable:
    > export TF_VAR_client_secret='12345678-1234-1234-1234-0123456789ab'

    > terraform plan

- Using a Variable file that contents the variable:

- Variable file: 

    File: my-azure-creds.tfvars
    ```hcl
    # Azure Credentials
    client_secret   = "12345678-1234-1234-1234-0123456789ab" // "password"
    ```

    > terraform apply -var-file="my-azure-creds.tfvars"

## TBD Create a Resource Group, Storage Account and Private Blob for tfplan file

Add to the main.tf, the creation of a Resource Group:
```hcl
resource "azurerm_resource_group" "terraform" {
  name = "my-saved-terraform-tfplan"
  location = "eastus"
}
```
Then run the plan.

    > terraform init
    > terraform plan -out=tfplan.out
    > terraform apply tfplan.out

<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
Error: no lines in file
<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
