# Lab: Variable Validation and Suppression

Duration: 15 minutes

We may want to validate and possibly suppress and sensitive information defined within our variables.

- Task 1: Validate variables in the Server Module
- Task 2: Suppress sensitive information
- Task 3: View the Terraform state file

## Task 1: Valdiate variables in the Server Module

Update the `server_os` variable within the server module's `variables.tf` configuration file to include a validation block to check for approved operating system types:

```hcl
variable "server_os" {
  type        = string
  description = "Server Operating System"
  default     = "ubuntu_20_04"

  validation {
    condition     = contains(["ubuntu_20_04", "ubuntu_18_04", "windows_2019"], lower(var.server_os))
    error_message = "You must use an approved operating system. Options are ubuntu_18_04, ubuntu_20_04, or windows_2019."
  }
}
```

Update the `server_os` variable in Terraform Cloud to list an approved server operating system and perform a `terraform plan`:

![Approved Server OS](./img/server_os_approved.png)


```bash
terraform plan
```

Update the `server_os` variable in Terraform Cloud to list an approved server operating system and perform a `terraform plan`: 

![Approved Server OS](./img/server_os_unapproved.png)

```bash
terraform plan
```

```bash
Initializing Terraform configuration...
╷
│ Error: Invalid value for variable
│ 
│   on main.tf line 47, in module "server":
│   47:   server_os              = var.server_os
│ 
│ You must use an approved operating system. Options are ubuntu_18_04,
│ ubuntu_20_04, or windows_2019.
│ 
│ This was checked by the validation rule at server/variables.tf:34,5-15.
```

## Task 2: Suppress sensitive information

Terraform allows us to mark variables as sensitive and suppress that information. Create `contactinfo.tf` and `terraform.auto.tfvars` files the `/workstation/terraform` root module directory.  

`contactinfo.tf`

```hcl
variable "department" {
    type = string

    validation {
        condition = length(var.department) == 3
        error_message = "Department length must be 3 characters."
    }
}

variable "cost_code" {
    type = string
    validation {
        condition = can(regex("^(?:[0-9]{1,2}\\-){2}[0-9]{1,2}$", var.cost_code))
        error_message = "Must be an cost code address of the form X-X-X."
    }
}

variable "phone_number" {
  type = string
  sensitive = true
  default = "867-5309"
}

locals {
  contact_info = {
      department = var.department
      cost_code = var.cost_code
      phone_number = var.phone_number
  }

  my_number = nonsensitive(var.phone_number)
}

output "department" {
  value = local.contact_info.department
}

output "cost_code" {
  value = local.contact_info.cost_code
}

output "phone_number" {
  value = local.contact_info.phone_number
}

output "my_number" {
  value = local.my_number
}
```

`terraform.auto.tfvars`

```hcl
department = "ABC"
cost_code  = "1-3-4"
```

Execute a `terraform apply` with the variables in the `terraform.auto.tfvars`.

```bash
terraform apply
```

You will notice that the output block errors as it needs to have the `sensitive = true` value set.

```bash
╷
│ Error: Output refers to sensitive values
│
│   on variables.tf line 73:
│   73: output "phone_number" {
│
│ To reduce the risk of accidentally exporting sensitive data that was intended to be only internal, Terraform requires that any root module output containing
│ sensitive data be explicitly marked as sensitive, to confirm your intent.
│
│ If you do intend to export this data, annotate the output value as sensitive by adding the following argument:
│     sensitive = true
╵
```

Update the output to set the `sensitive = true` attribute and rerun the apply.

```hcl
output "phone_number" {
  sensitive = true
  value = local.contact_info.phone_number
}
```

```bash
terraform apply
```

```bash
Outputs:

cost_code = "1-3-4"
department = "ABC"
my_number = "867-5309"
phone_number = <sensitive>
```

## Task 4: View the Terraform State File

Even though items are marked as sensitive within the Terraform configuration, they are stored within the Terraform state file. It is therefore critical to limit the access to the Terraform state file.

View the most current terraform state file on the `States` tab within Terraform Cloud

![Sensitive State](./img/sensitive_state.png)
```json
{
 {
  "version": 4,
  "terraform_version": "1.0.9",
  "serial": 5,
  "lineage": "67c71c6f-af25-b300-2ea7-6d0f4b226b7b",
  "outputs": {
    "cost_code": {
      "value": "1-3-4",
      "type": "string"
    },
    "department": {
      "value": "ABC",
      "type": "string"
    },
    "my_number": {
      "value": "867-5309",
      "type": "string"
    },
    "phone_number": {
      "value": "867-5309",
      "type": "string",
      "sensitive": true
    },
    ...
```
