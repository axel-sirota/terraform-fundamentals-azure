# Instalation instructions

## Windows

### Core requirements:

**Azure CLI**:

- The main steps are from https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-windows?tabs=azure-cli

1) Download and execute the installer from the following link: https://aka.ms/installazurecliwindows

2) Authenticate executing the command:

`az login`

You should get as output your subscription information (or all your subscriptions if many).

If you happen to have an error euthenticating this way, or you need to change subscriptions jus run:

`az login --tenant {YOUR SUBSCRIPTION ID}`

**Git Bash**

1) Download from this link: https://github.com/git-for-windows/git/releases/download/v2.39.1.windows.1/Git-2.39.1-64-bit.exe

2) Install following the instructions.

**Terraform**


1) The best way is to use Chocolatey (https://chocolatey.org/install) and run:

`choco install terraform`

OR

1') But if you dont have nor use Chocolatey then first download from this link: https://releases.hashicorp.com/terraform/1.3.7/terraform_1.3.7_windows_amd64.zip

2) Extract into a new folder in your C drive (`c:\terraform`)

3) Add to PATH:

    - Select Control Panel and then System.
    - Click Advanced and then Environment Variables.
    - Double click the PATH variable
    - Add a new entry `c:\terraform`

4) Test by opening a new terminal and run:

`terraform -version`

6) You are set.

### How to check if everything works?

1) Create a directory in your User folder, let's call it tmp.

`mkdir tmp`

2) Create a file `providers.tf` with the following content

```
terraform {
  required_version = ">=0.12"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~>2.0"
    }
    random = {
      source  = "hashicorp/random"
      version = "~>3.0"
    }
  }
}

provider "azurerm" {
  features {}
}
```

3) Create a file `variables.tf` with the following content

```
variable "resource_group_location" {
  default     = "eastus"
  description = "Location of the resource group."
}

variable "resource_group_name_prefix" {
  default     = "rg"
  description = "Prefix of the resource group name that's combined with a random ID so name is unique in your Azure subscription."
}
```

4) Create a file `outputs.tf` with the following content

```
output "resource_group_name" {
  value = azurerm_resource_group.rg.name
}
```

5) Finally, create a file names `main.tf` with the following content:

```
resource "random_pet" "rg_name" {
  prefix = var.resource_group_name_prefix
}

resource "azurerm_resource_group" "rg" {
  location = var.resource_group_location
  name     = random_pet.rg_name.id
}
```

6) Run `terraform init` to initialize the providers

7) Run `terraform plan`, it should display that is going to create 2 resources (a random resource string and an actual resource group in azure)

8) Run `terraform apply`.

9) Verify it was OK because after applying the following command:

`terraform output resource_group_name`

You get that name and run:

`az group show --name {NAME YOU GOT ABOVE}`

and it returns a resource.

10) Cleanup by running `terraform destroy`

