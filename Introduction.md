# Terraform
### Terraform is in huge demand these days and it is basically in demand because an enterprize want to avoid
 Hire Multiple talent for each cloud provider
 Want to keep things simple and sharable
 Want to avoid vendor lockin
 Want to evaluate multiple cloud provider for there business

## Terraform is open source tool developed by the Harshicorp and it is also be used for Infrastructure as a code (IAC)
## It uses the declarative language hcl (Harshicorp configuration language) to write a code 
## Best way to standardize the infrastructure
## Best tool for multiple cloud infra and we can test the entire infra for out code to run
## It is a sharable file and used to share across the team
## Uses different providers for building the complete tf file for managing the infra & deployment of infra

# Workflow for Terraform
### Terraform is same like building the infra in cloud from GUI, but it is like doing all at once
    1. Write - Writing the code to build multiple types of required resources in cloud
    2. Plan - Review what has been written and revisit as it is associated with cost
    3. Apply - Final run , as it will immidiately create the entire resoorces inside the provided cloud destination like Azure or AWS or Google Cloud


# Installation of Terraform in window machine
    Step 1.  Download terraform from its harshicorp provider, better to download latest stable version
    Step 2. Extract the zip file downloaded and then simply put it in C:\terraform folder, create folder if not exist in c drive
    Step 3. Get the system environment path modified so that simple command run like terraform --v

# If we are using azure and we already have subscription then we need these details for sure
##  client_id, client_secret, tenant_id, subscription_id
### Client Id and Client Secret can be created by going to Azure Portal & then search for Microsoft Entra Id, and then Manage --> App registration  --> New Registration --> Give the Desired Name --> Click Register
### SPN --> Manage --> Certificate & Secrets --> Client Secret --> New Client Secret --> Add --> Copy Value and store it somewhere safe

## Refer Terraform documentation
    https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs
    https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group


        terraform init
        terraform plan -out main.tfplan
        terraform apply



            resource "azurerm_resource_group" "rgdetails" {
              name     = "rg-tutorial"
              location = "Australia East"
            }

            resource "azurerm_storage_account" "storagedetails" {
              name                     = "storagetutorialtrform2222026"
              resource_group_name      = azurerm_resource_group.rgdetails.name
              location                 = azurerm_resource_group.rgdetails.location
              account_tier             = "Standard"
              account_replication_type = "LRS"
              account_kind = "StorageV2"
            }

            resource "azurerm_storage_container" "storagecontainer" {
              name                  = "data"
              storage_account_id    = azurerm_storage_account.storagedetails.id
              container_access_type = "blob"
            }

            resource "azurerm_storage_blob" "storageblob" {
              name                   = "readme.txt"
              storage_account_name   = azurerm_storage_account.storagedetails.name
              storage_container_name = azurerm_storage_container.storagecontainer.name
              type                   = "Block"
              source                 = "README.md"
            }

# Be Cautious, Destroy command will simply delete every thing in one shot
terraform destroy
### Always delete resources with terraform when it is cteated by terraform to make sure it is not giving any issue while recreating

    All the work we have done till now is without dependency, now we must declare the dependency, it has to be there to make sure terraform know the exact dependecy of the resource sequence


            resource "azurerm_resource_group" "rgdetails" {
              name     = "rg-tutorial"
              location = "Australia East"
            }

            resource "azurerm_storage_account" "storagedetails" {
              name                     = "strgtutorialtrform2026"
              resource_group_name      = azurerm_resource_group.rgdetails.name
              location                 = azurerm_resource_group.rgdetails.location
              account_tier             = "Standard"
              account_replication_type = "LRS"
              account_kind = "StorageV2"

              depends_on = [ azurerm_resource_group.rgdetails ]
            }

            resource "azurerm_storage_container" "storagecontainer" {
              name                  = "data"
              storage_account_id    = azurerm_storage_account.storagedetails.id
              container_access_type = "blob"

              depends_on = [ azurerm_storage_account.storagedetails ]
            }

            resource "azurerm_storage_blob" "storageblob" {
              name                   = "readme.txt"
              storage_account_name   = azurerm_storage_account.storagedetails.name
              storage_container_name = azurerm_storage_container.storagecontainer.name
              type                   = "Block"
              source                 = "README.md"

              depends_on = [ azurerm_storage_container.storagecontainer ]
            }


Not refer the next steps or tutorial in tutorial1.md