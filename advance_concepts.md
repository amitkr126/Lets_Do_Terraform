# Terraform is also like general progeamming language where we can have many logics like loop to ease out complexity
        terraform init -upgrade

## HCL is awesome for all the providers and it has an awesome tutorials, however we can learn things only by doing it and practicing it well
    Power of Foreach and variable

### Let us learn some advance concepts, i am using azure as provider here but we can also use AWS & GCP during out coming tutorials

    Lets we need to crteate multiple resources like Resource Group, Storage account, Virtual network then we can do it with loops

        # 2. Variable Definition
        variable "infrastructure_configs" {
          type = map(object({
            location      = string
            rg_name       = string
            vnet_name     = string
            address_space = list(string)
            storage_name  = string
          }))

          default = {
            "marketing" = {
              location      = "East US"
              rg_name       = "rg-marketing-prod"
              vnet_name     = "vnet-mkt"
              address_space = ["10.10.0.0/16"]
              storage_name  = "stmktdat" # Suffix added later for uniqueness
            },
            "finance" = {
              location      = "West Europe"
              rg_name       = "rg-finance-prod"
              vnet_name     = "vnet-fin"
              address_space = ["10.20.0.0/16"]
              storage_name  = "stfindat"
            }
          }
        }

        # 3. Random String (To ensure unique storage names)
        resource "random_string" "suffix" {
          for_each = var.infrastructure_configs
          length   = 6
          special  = false
          upper    = false
        }

        # 4. Resource Group
        resource "azurerm_resource_group" "rg" {
          for_each = var.infrastructure_configs

          name     = each.value.rg_name
          location = each.value.location
        }

        # 5. Virtual Network
        resource "azurerm_virtual_network" "vnet" {
          for_each = var.infrastructure_configs

          name                = each.value.vnet_name
          location            = azurerm_resource_group.rg[each.key].location
          resource_group_name = azurerm_resource_group.rg[each.key].name
          address_space       = each.value.address_space
        }

        # 6. Storage Account
        resource "azurerm_storage_account" "storage" {
          for_each = var.infrastructure_configs

          # Concatenating name + random string to avoid global naming conflicts
          name                     = "${each.value.storage_name}${random_string.suffix[each.key].result}"
          resource_group_name      = azurerm_resource_group.rg[each.key].name
          location                 = azurerm_resource_group.rg[each.key].location
          account_tier             = "Standard"
          account_replication_type = "LRS"
        }