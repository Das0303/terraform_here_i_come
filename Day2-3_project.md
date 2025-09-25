## Project : Create a VM using terraform


Main.tf
_____________________________

provider "azurerm" {
  features {}
  subscription_id = "8866bd54-6b9d-4bf5-975a-4df57b64cb4d"
}

resource "azurerm_resource_group" "RG" {
  name     = "${var.prefix}-RG"
  location = "West Europe"
}

resource "azurerm_virtual_network" "Nw" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.RG.location
  resource_group_name = azurerm_resource_group.RG.name
}

resource "azurerm_subnet" "snet" {
  name                 = "snet"
  resource_group_name  = azurerm_resource_group.RG.name
  virtual_network_name = azurerm_virtual_network.Nw.name
  address_prefixes     = ["10.0.2.0/24"]
}

resource "azurerm_network_interface" "interface" {
  name                = "${var.prefix}-nic"
  location            = azurerm_resource_group.RG.location
  resource_group_name = azurerm_resource_group.RG.name

  ip_configuration {
    name                          = "testconfiguration1"
    subnet_id                     = azurerm_subnet.snet.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_virtual_machine" "main" {
  name                  = "${var.prefix}-vm"
  location              = azurerm_resource_group.RG.location
  resource_group_name   = azurerm_resource_group.RG.name
  network_interface_ids = [azurerm_network_interface.interface.id]
  vm_size               = "Standard_B1s"

  # Uncomment this line to delete the OS disk automatically when deleting the VM
  delete_os_disk_on_termination = true

  # Uncomment this line to delete the data disks automatically when deleting the VM
  delete_data_disks_on_termination = true

  storage_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  storage_os_disk {
    name              = "myosdisk1"
    caching           = "ReadWrite"
    create_option     = "FromImage"
    managed_disk_type = "Standard_LRS"
  }
  os_profile {
    computer_name  = "hostname"
    admin_username = "testadmin"
    admin_password = "Password1234!"
  }
  os_profile_linux_config {
    disable_password_authentication = false
  }
  tags = {
    environment = "staging"
  }
}



variables.tf
______________________

variable "prefix" {
  default = "tf"
}


## results :
______________

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform plan
╷
│ Error: Reference to undeclared resource
│
│   on main.tf line 4, in resource "azurerm_virtual_network" "Nw":
│    4:   location            = azurerm_resource_group.RG.location
│
│ A managed resource "azurerm_resource_group" "RG" has not been declared in the root module.  
╵
╷
│ Error: Reference to undeclared resource
│
│   on main.tf line 5, in resource "azurerm_virtual_network" "Nw":
│    5:   resource_group_name = azurerm_resource_group.RG.name
│
│ A managed resource "azurerm_resource_group" "RG" has not been declared in the root module.  
╵

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform plan
╷
│ Error: Reference to undeclared resource
│
│   on main.tf line 4, in resource "azurerm_virtual_network" "Nw":
│    4:   location            = azurerm_resource_group.RG.location
│
│ A managed resource "azurerm_resource_group" "RG" has not been declared in the root module.  
╵
╷
│ Error: Reference to undeclared resource
│
│   on main.tf line 5, in resource "azurerm_virtual_network" "Nw":
│    5:   resource_group_name = azurerm_resource_group.RG.name
│
│ A managed resource "azurerm_resource_group" "RG" has not been declared in the root module.  
╵

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform plan

Planning failed. Terraform encountered an error while generating this plan.

╷
│ Error: Invalid provider configuration
│
│ Provider "registry.terraform.io/hashicorp/azurerm" requires explicit configuration. Add a   
│ provider block to the root module and configure the provider's required arguments as        
│ described in the provider documentation.
│
╵
╷
│ Error: Missing required argument
│
│   with provider["registry.terraform.io/hashicorp/azurerm"],
│   on <empty> line 0:
│   (source code not available)
│
│ The argument "features" is required, but no definition was found.
╵

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform init
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Using previously-installed hashicorp/azurerm v4.45.1

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform plan

Planning failed. Terraform encountered an error while generating this plan.

╷
│ Error: building account: unable to configure ResourceManagerAccount: subscription ID could not be determined and was not specified
│
│   with provider["registry.terraform.io/hashicorp/azurerm"],
│   on main.tf line 1, in provider "azurerm":
│    1: provider "azurerm" {
│
╵

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ az account show
{
  "environmentName": "AzureCloud",
  "homeTenantId": "078494c6-36c6-4b10-9294-78bea291f6d6",
  "id": "8866bd54-6b9d-4bf5-975a-4df57b64cb4d",
  "isDefault": true,
  "managedByTenants": [],
  "name": "Subscription 1",
  "state": "Enabled",
  "tenantDefaultDomain": "ayusmitadas3gmail.onmicrosoft.com",
  "tenantDisplayName": "Default Directory",
  "tenantId": "078494c6-36c6-4b10-9294-78bea291f6d6",
  "user": {
    "name": "ayusmitadas3@gmail.com",
    "type": "user"
  }
}

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform init
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Using previously-installed hashicorp/azurerm v4.45.1

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform plan

Terraform used the selected providers to generate the following execution plan. Resource      
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_network_interface.interface will be created
  + resource "azurerm_network_interface" "interface" {
      + accelerated_networking_enabled = false
      + applied_dns_servers            = (known after apply)
      + id                             = (known after apply)
      + internal_domain_name_suffix    = (known after apply)
      + ip_forwarding_enabled          = false
      + location                       = "westeurope"
      + mac_address                    = (known after apply)
      + name                           = "tf-nic"
      + private_ip_address             = (known after apply)
      + private_ip_addresses           = (known after apply)
      + resource_group_name            = "tf-RG"
      + virtual_machine_id             = (known after apply)

      + ip_configuration {
          + gateway_load_balancer_frontend_ip_configuration_id = (known after apply)
          + name                                               = "testconfiguration1"
          + primary                                            = (known after apply)
          + private_ip_address                                 = (known after apply)
          + private_ip_address_allocation                      = "Dynamic"
          + private_ip_address_version                         = "IPv4"
          + subnet_id                                          = (known after apply)
        }
    }

  # azurerm_resource_group.RG will be created
  + resource "azurerm_resource_group" "RG" {
      + id       = (known after apply)
      + location = "westeurope"
      + name     = "tf-RG"
    }

  # azurerm_subnet.snet will be created
  + resource "azurerm_subnet" "snet" {
      + address_prefixes                              = [
          + "10.0.2.0/24",
        ]
      + default_outbound_access_enabled               = true
      + id                                            = (known after apply)
      + name                                          = "snet"
      + private_endpoint_network_policies             = "Disabled"
      + private_link_service_network_policies_enabled = true
      + resource_group_name                           = "tf-RG"
      + virtual_network_name                          = "tf-network"
    }

  # azurerm_virtual_machine.main will be created
  + resource "azurerm_virtual_machine" "main" {
      + availability_set_id              = (known after apply)
      + delete_data_disks_on_termination = true
      + delete_os_disk_on_termination    = true
      + id                               = (known after apply)
      + license_type                     = (known after apply)
      + location                         = "westeurope"
      + name                             = "tf-vm"
      + network_interface_ids            = (known after apply)
      + resource_group_name              = "tf-RG"
      + tags                             = {
          + "environment" = "staging"
        }
      + vm_size                          = "Standard_B1s"

      + os_profile {
          # At least one attribute in this block is (or was) sensitive,
          # so its contents will not be displayed.
        }

      + os_profile_linux_config {
          + disable_password_authentication = false
        }

      + storage_data_disk (known after apply)

      + storage_image_reference {
            id        = null
          + offer     = "0001-com-ubuntu-server-jammy"
          + publisher = "Canonical"
          + sku       = "22_04-lts"
          + version   = "latest"
        }

      + storage_os_disk {
          + caching                   = "ReadWrite"
          + create_option             = "FromImage"
          + disk_size_gb              = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = "Standard_LRS"
          + name                      = "myosdisk1"
          + os_type                   = (known after apply)
          + write_accelerator_enabled = false
        }
    }

  # azurerm_virtual_network.Nw will be created
  + resource "azurerm_virtual_network" "Nw" {
      + address_space                  = [
          + "10.0.0.0/16",
        ]
      + dns_servers                    = (known after apply)
      + guid                           = (known after apply)
      + id                             = (known after apply)
      + location                       = "westeurope"
      + name                           = "tf-network"
      + private_endpoint_vnet_policies = "Disabled"
      + resource_group_name            = "tf-RG"
      + subnet                         = (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take  
exactly these actions if you run "terraform apply" now.

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource      
actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # azurerm_network_interface.interface will be created
  + resource "azurerm_network_interface" "interface" {
      + accelerated_networking_enabled = false
      + applied_dns_servers            = (known after apply)
      + id                             = (known after apply)
      + internal_domain_name_suffix    = (known after apply)
      + ip_forwarding_enabled          = false
      + location                       = "westeurope"
      + mac_address                    = (known after apply)
      + name                           = "tf-nic"
      + private_ip_address             = (known after apply)
      + private_ip_addresses           = (known after apply)
      + resource_group_name            = "tf-RG"
      + virtual_machine_id             = (known after apply)

      + ip_configuration {
          + gateway_load_balancer_frontend_ip_configuration_id = (known after apply)
          + name                                               = "testconfiguration1"
          + primary                                            = (known after apply)
          + private_ip_address                                 = (known after apply)
          + private_ip_address_allocation                      = "Dynamic"
          + private_ip_address_version                         = "IPv4"
          + subnet_id                                          = (known after apply)
        }
    }

  # azurerm_resource_group.RG will be created
  + resource "azurerm_resource_group" "RG" {
      + id       = (known after apply)
      + location = "westeurope"
      + name     = "tf-RG"
    }

  # azurerm_subnet.snet will be created
  + resource "azurerm_subnet" "snet" {
      + address_prefixes                              = [
          + "10.0.2.0/24",
        ]
      + default_outbound_access_enabled               = true
      + id                                            = (known after apply)
      + name                                          = "snet"
      + private_endpoint_network_policies             = "Disabled"
      + private_link_service_network_policies_enabled = true
      + resource_group_name                           = "tf-RG"
      + virtual_network_name                          = "tf-network"
    }

  # azurerm_virtual_machine.main will be created
  + resource "azurerm_virtual_machine" "main" {
      + availability_set_id              = (known after apply)
      + delete_data_disks_on_termination = true
      + delete_os_disk_on_termination    = true
      + id                               = (known after apply)
      + license_type                     = (known after apply)
      + location                         = "westeurope"
      + name                             = "tf-vm"
      + network_interface_ids            = (known after apply)
      + resource_group_name              = "tf-RG"
      + tags                             = {
          + "environment" = "staging"
        }
      + vm_size                          = "Standard_B1s"

      + os_profile {
          # At least one attribute in this block is (or was) sensitive,
          # so its contents will not be displayed.
        }

      + os_profile_linux_config {
          + disable_password_authentication = false
        }

      + storage_data_disk (known after apply)

      + storage_image_reference {
            id        = null
          + offer     = "0001-com-ubuntu-server-jammy"
          + publisher = "Canonical"
          + sku       = "22_04-lts"
          + version   = "latest"
        }

      + storage_os_disk {
          + caching                   = "ReadWrite"
          + create_option             = "FromImage"
          + disk_size_gb              = (known after apply)
          + managed_disk_id           = (known after apply)
          + managed_disk_type         = "Standard_LRS"
          + name                      = "myosdisk1"
          + os_type                   = (known after apply)
          + write_accelerator_enabled = false
        }
    }

  # azurerm_virtual_network.Nw will be created
  + resource "azurerm_virtual_network" "Nw" {
      + address_space                  = [
          + "10.0.0.0/16",
        ]
      + dns_servers                    = (known after apply)
      + guid                           = (known after apply)
      + id                             = (known after apply)
      + location                       = "westeurope"
      + name                           = "tf-network"
      + private_endpoint_vnet_policies = "Disabled"
      + resource_group_name            = "tf-RG"
      + subnet                         = (known after apply)
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

azurerm_resource_group.RG: Creating...
azurerm_resource_group.RG: Still creating... [00m10s elapsed]
azurerm_resource_group.RG: Creation complete after 16s [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG]
azurerm_virtual_network.Nw: Creating...
azurerm_virtual_network.Nw: Creation complete after 9s [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network] 
azurerm_subnet.snet: Creating...
azurerm_subnet.snet: Creation complete after 10s [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network/subnets/snet]
azurerm_network_interface.interface: Creating...
azurerm_network_interface.interface: Still creating... [00m10s elapsed]
azurerm_network_interface.interface: Creation complete after 15s [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/networkInterfaces/tf-nic]
azurerm_virtual_machine.main: Creating...
azurerm_virtual_machine.main: Still creating... [00m10s elapsed]
azurerm_virtual_machine.main: Still creating... [00m20s elapsed]
azurerm_virtual_machine.main: Creation complete after 24s [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Compute/virtualMachines/tf-vm]   

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

AYUSH@AYUSMITA MINGW64 /d/DevopsCourse/TerraForm (master)
$ terraform destroy
azurerm_resource_group.RG: Refreshing state... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG]
azurerm_virtual_network.Nw: Refreshing state... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network]        
azurerm_subnet.snet: Refreshing state... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network/subnets/snet]  
azurerm_network_interface.interface: Refreshing state... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/networkInterfaces/tf-nic] 
azurerm_virtual_machine.main: Refreshing state... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Compute/virtualMachines/tf-vm]

Terraform used the selected providers to generate the following execution plan. Resource      
actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # azurerm_network_interface.interface will be destroyed
  - resource "azurerm_network_interface" "interface" {
      - accelerated_networking_enabled = false -> null
      - applied_dns_servers            = [] -> null
      - dns_servers                    = [] -> null
      - id                             = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/networkInterfaces/tf-nic" -> null
      - internal_domain_name_suffix    = "mer4l5suupgehjvdhxhsyebuae.ax.internal.cloudapp.net"
 -> null
      - ip_forwarding_enabled          = false -> null
      - location                       = "westeurope" -> null
      - mac_address                    = "60-45-BD-A1-87-46" -> null
      - name                           = "tf-nic" -> null
      - private_ip_address             = "10.0.2.4" -> null
      - private_ip_addresses           = [
          - "10.0.2.4",
        ] -> null
      - resource_group_name            = "tf-RG" -> null
      - tags                           = {} -> null
      - virtual_machine_id             = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Compute/virtualMachines/tf-vm" -> null
        # (4 unchanged attributes hidden)

      - ip_configuration {
          - name                                               = "testconfiguration1" -> null 
          - primary                                            = true -> null
          - private_ip_address                                 = "10.0.2.4" -> null
          - private_ip_address_allocation                      = "Dynamic" -> null
          - private_ip_address_version                         = "IPv4" -> null
          - subnet_id                                          = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network/subnets/snet" -> null
            # (2 unchanged attributes hidden)
        }
    }

  # azurerm_resource_group.RG will be destroyed
  - resource "azurerm_resource_group" "RG" {
      - id         = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG" -> null
      - location   = "westeurope" -> null
      - name       = "tf-RG" -> null
      - tags       = {} -> null
        # (1 unchanged attribute hidden)
    }

  # azurerm_subnet.snet will be destroyed
  - resource "azurerm_subnet" "snet" {
      - address_prefixes                              = [
          - "10.0.2.0/24",
        ] -> null
      - default_outbound_access_enabled               = true -> null
      - id                                            = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network/subnets/snet" -> null
      - name                                          = "snet" -> null
      - private_endpoint_network_policies             = "Disabled" -> null
      - private_link_service_network_policies_enabled = true -> null
      - resource_group_name                           = "tf-RG" -> null
      - service_endpoint_policy_ids                   = [] -> null
      - service_endpoints                             = [] -> null
      - virtual_network_name                          = "tf-network" -> null
        # (1 unchanged attribute hidden)
    }

  # azurerm_virtual_machine.main will be destroyed
  - resource "azurerm_virtual_machine" "main" {
      - delete_data_disks_on_termination = true -> null
      - delete_os_disk_on_termination    = true -> null
      - id                               = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Compute/virtualMachines/tf-vm" -> null
      - location                         = "westeurope" -> null
      - name                             = "tf-vm" -> null
      - network_interface_ids            = [
          - "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/networkInterfaces/tf-nic",
        ] -> null
      - resource_group_name              = "tf-RG" -> null
      - tags                             = {
          - "environment" = "staging"
        } -> null
      - vm_size                          = "Standard_B1s" -> null
      - zones                            = [] -> null

      - os_profile {
          # At least one attribute in this block is (or was) sensitive,
          # so its contents will not be displayed.
        }

      - os_profile_linux_config {
          - disable_password_authentication = false -> null
        }

      - storage_image_reference {
            id        = null
          - offer     = "0001-com-ubuntu-server-jammy" -> null
          - publisher = "Canonical" -> null
          - sku       = "22_04-lts" -> null
          - version   = "latest" -> null
        }

      - storage_os_disk {
          - caching                   = "ReadWrite" -> null
          - create_option             = "FromImage" -> null
          - disk_size_gb              = 30 -> null
          - managed_disk_id           = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Compute/disks/myosdisk1" -> null
          - managed_disk_type         = "Standard_LRS" -> null
          - name                      = "myosdisk1" -> null
          - os_type                   = "Linux" -> null
          - write_accelerator_enabled = false -> null
            # (2 unchanged attributes hidden)
        }
    }

  # azurerm_virtual_network.Nw will be destroyed
  - resource "azurerm_virtual_network" "Nw" {
      - address_space                  = [
          - "10.0.0.0/16",
        ] -> null
      - dns_servers                    = [] -> null
      - flow_timeout_in_minutes        = 0 -> null
      - guid                           = "fee52361-a354-43cc-a6a3-3dcf2c103404" -> null       
      - id                             = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network" -> null
      - location                       = "westeurope" -> null
      - name                           = "tf-network" -> null
      - private_endpoint_vnet_policies = "Disabled" -> null
      - resource_group_name            = "tf-RG" -> null
      - subnet                         = [
          - {
              - address_prefixes                              = [
                  - "10.0.2.0/24",
                ]
              - default_outbound_access_enabled               = true
              - delegation                                    = []
              - id                                            = "/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network/subnets/snet"
              - name                                          = "snet"
              - private_endpoint_network_policies             = "Disabled"
              - private_link_service_network_policies_enabled = true
              - service_endpoint_policy_ids                   = []
              - service_endpoints                             = []
                # (2 unchanged attributes hidden)
            },
        ] -> null
      - tags                           = {} -> null
        # (2 unchanged attributes hidden)
    }

Plan: 0 to add, 0 to change, 5 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

azurerm_virtual_machine.main: Destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Compute/virtualMachines/tf-vm]
azurerm_virtual_machine.main: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...icrosoft.Compute/virtualMachines/tf-vm, 00m10s elapsed]
azurerm_virtual_machine.main: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...icrosoft.Compute/virtualMachines/tf-vm, 00m20s elapsed]
azurerm_virtual_machine.main: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...icrosoft.Compute/virtualMachines/tf-vm, 00m30s elapsed]
azurerm_virtual_machine.main: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...icrosoft.Compute/virtualMachines/tf-vm, 00m40s elapsed]
azurerm_virtual_machine.main: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...icrosoft.Compute/virtualMachines/tf-vm, 00m50s elapsed]
azurerm_virtual_machine.main: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...icrosoft.Compute/virtualMachines/tf-vm, 01m00s elapsed]
azurerm_virtual_machine.main: Destruction complete after 1m0s
azurerm_network_interface.interface: Destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/networkInterfaces/tf-nic]       
azurerm_network_interface.interface: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...osoft.Network/networkInterfaces/tf-nic, 00m10s elapsed]
azurerm_network_interface.interface: Destruction complete after 13s
azurerm_subnet.snet: Destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network/subnets/snet]        
azurerm_subnet.snet: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...irtualNetworks/tf-network/subnets/snet, 00m10s elapsed]
azurerm_subnet.snet: Destruction complete after 13s
azurerm_virtual_network.Nw: Destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG/providers/Microsoft.Network/virtualNetworks/tf-network]
azurerm_virtual_network.Nw: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-...oft.Network/virtualNetworks/tf-network, 00m10s elapsed]
azurerm_virtual_network.Nw: Destruction complete after 12s
azurerm_resource_group.RG: Destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG]
azurerm_resource_group.RG: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG, 00m10s elapsed]
azurerm_resource_group.RG: Still destroying... [id=/subscriptions/8866bd54-6b9d-4bf5-975a-4df57b64cb4d/resourceGroups/tf-RG, 00m20s elapsed]
azurerm_resource_group.RG: Destruction complete after 22s

Destroy complete! Resources: 5 destroyed.



## Analysis:

1. if you notice i kept doing mistakes of not saving the file main.tf after any updates was done. That comes out to be one of the common yet effective mistake as a beginner
anyone would do. So saving the files is always recommended

2. i had declared the reosurce group module but i had invoked it in my later code . which threw and error and i had to resolve it.

3. i had declared the provider block in the start , this fumbled the terraform to install the required plugins for processing my code.

4. always run terrafrom init , plan and apply in the order.. this way you are allowing terraform to process your code step by step and helps you identify any issues and help resolve it.


