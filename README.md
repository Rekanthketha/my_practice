# my_practice
terraform {
    required_providers{
        azurerm = {
            source = "hashicorp/azurerm"
            version = "=2.46.0"
        }
    }
}

provider "azurerm" {
    features {}
    subscription_id = var.subscription_id
    client_id = var.client_id
    client_secret = var.client_secret
    tenant_id = var.tenant_id
}



resource "azurerm_resource_group" "rk-rs" { 
    name = "rekanth-resource"
    location = "East US"
}

resource "azurerm_virtual_network" "rk-vn" {
    name = "rekanth-vnet"
    resource_group_name = azurerm_resource_group.rk-rs.name
    location = azurerm_resource_group.rk-rs.location
    address_space = ["10.70.0.0/16"]
    
}

resource "azurerm_subnet" "rk-subnet"{
name = "rekanth-subnet"

..............................
resource_group_name = azurerm_resource_group.rk-rs.name
virtual_network_name = azurerm_virtual_network.rk-vn.name
address_prefixes = ["10.70.1.0/24"]

}

resource "azurerm_public_ip" "rk-p_ip" {
name = "rekanth-piblic_ip"
resource_group_name = azurerm_resource_group.rk-rs.name
location = azurerm_resource_group.rk-rs.location
allocation_method = "Static"


}

resource "azurerm_network_interface" "rk-if"{
name = "rekanth-interface"
resource_group_name = azurerm_resource_group.rk-rs.name
location = azurerm_resource_group.rk-rs.location

         ip_configuration {
             name ="public"
             subnet_id = azurerm_subnet.rk-subnet.id
             private_ip_address_allocation = "Dynamic"
             public_ip_address_id =azurerm_public_ip.rk-p_ip.id
             


     }
}

resource "azurerm_linux_virtual_machine" "webserverlabel"{
name = "webserver"
resource_group_name = azurerm_resource_group.rk-rs.name
location = azurerm_resource_group.rk-rs.location
size = "Standard_F2"
admin_username = "adminuser"
network_interface_ids = [azurerm_network_interface.rk-if.id
                         ]



admin_ssh_key {
    username = "adminuser"
    public_key = file("~/.ssh/id_rsa.pub")
}

os_disk{
    caching = "ReadWrite"
    storage_account_type = "Standard_LRS"
}

source_image_reference {
    publisher = "Canonical"
    offer = "UbuntuServer"
    sku = "18.04-LTS"
    version = "latest"
}
}

