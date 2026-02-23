# Terraform Tutorial 1 - The Next Step

## Refer Terraform documentation
    https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs
    https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/resource_group



        terraform init
        terraform plan -out main.tfplan
        terraform apply  or terraform apply main.tfplan
        terraform destroy

## Virtual Network creation and Sunet Creation
         terraform {
          required_providers {
            azurerm = {
              source = "hashicorp/azurerm"
              version = "4.61.0"
            }
          }
        }

      provider "azurerm" {
          features {}
  
          client_id       = "d43e93a0-xxx-xxxx-xxxx-xxxxxxx"
          client_secret   = "qgB8Q~xxxxxxxx.xxxxxxx-xxxxxxx"
          tenant_id       = "87e219e4-xxxx-xxxx-xxxx-xxxxxxxxx"
          subscription_id = "81a16583-xxxx-xxxx-xxxxx-xxxxxx"
  
      }

        locals {
            resource_group_name  = "rg-secured"
            location = "Australia East"

            virtual_network = {
                name = "vnet-22222026" 
                address_space = ["10.0.0.0/16"]

            }

            subnet =[
               {
                    name             = "app-subnet"
                    address_prefixes = ["10.0.1.0/24"]
               },
               {
                    name             = "db-subnet"
                    address_prefixes = ["10.0.2.0/24"]
               }

            ] 
        }

        resource "azurerm_resource_group" "rgdetails" {
          name     = local.resource_group_name
          location = local.location
        }

        resource "azurerm_virtual_network" "vnetdetail" {
          name                = local.virtual_network.name
          location            = local.location
          resource_group_name = local.resource_group_name
          address_space       = local.virtual_network.address_space
          dns_servers         = ["10.0.0.4", "10.0.0.5"]

          depends_on = [ azurerm_resource_group.rgdetails ]
        }

        resource "azurerm_subnet" "appsubnet" {
          name                 = local.subnet[0].name
          resource_group_name  = local.resource_group_name
          virtual_network_name = local.virtual_network.name
          address_prefixes     = local.subnet[0].address_prefixes

          depends_on = [ azurerm_virtual_network.vnetdetail ]
        }
     
        resource "azurerm_network_interface" "nicdetails" {
          name                = "tutorial01-nic"
          location            = local.location
          resource_group_name = local.resource_group_name

          ip_configuration {
            name                          = "internal"
            subnet_id                     = azurerm_subnet.appsubnet.id
            private_ip_address_allocation = "Dynamic"
            public_ip_address_id =  azurerm_public_ip.pipdetails.id

          }

          depends_on =  [ azurerm_subnet.appsubnet ]
        }

        resource "azurerm_public_ip" "pipdetails" {
          name                = "tutorialPublicIp-0001"
          resource_group_name = local.resource_group_name
          location            = local.location
          allocation_method   = "Static"

          depends_on = [ azurerm_resource_group.rgdetails ]

        }

        resource "azurerm_network_security_group" "nsgroup" {
          name                = "tutorialsg-0001"
          location            = local.location
          resource_group_name = local.resource_group_name
        
          security_rule {
            name                       = "allow-RDP"
            priority                   = 100
            direction                  = "Inbound"
            access                     = "Allow"
            protocol                   = "Tcp"
            source_port_range          = "*"
            destination_port_range     = "3389"
            source_address_prefix      = "*"
            destination_address_prefix = "*"
          }
        
          depends_on = [ azurerm_resource_group.rgdetails ]
        
        }
        
        resource "azurerm_subnet_network_security_group_association" "nsglink" {
          subnet_id                 = azurerm_subnet.appsubnet.id
          network_security_group_id = azurerm_network_security_group.nsgroup.id
        }
        
        
        
        output "appsubnet-id" {
          value = azurerm_subnet.appsubnet.id
        }



### Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
## Outputs:
    appsubnet-id = "/subscriptions/81a16583-cae3-4e5b-9aaf-d18265b1cdfd/resourceGroups/rg-secured/providers/Microsoft.Network/virtualNetworks/vnet-22222026/subnets/app-subnet"
