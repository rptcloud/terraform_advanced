# Lab: Terraform Getting Started

In this lab you will deploy a base Terraform configuration containing a local module

Duration: 15 minutes

- Task 1: Connect to class workstation
- Task 2: Create Terraform Configuration
- Task 3: Create Terraform Module
- Task 4: Validate Terraform
- Task 5: Deploy Infrastructure

## Task 1: Log into Class workstation

Navigate to your lab workstation using the URL provide by your instructor. Log into the workstation using the password provided by your instructor.

Open Folder to `/workstation/terraform` and validate that you can navigate to the included `main.tf` and `terraform.tfvars`

If desired, enable the VSCode Terraform extension by browsing to the extensions icon on the left menu bar and search for `Terraform`. Install the `HashiCorp Terraform` extension.

![](/img/terraform_extension.png)

Install the extension and after it is installed refresh your browser to activate.

## Task 2: Create Terraform Configuration

Replace the contents of `main.tf` with the following terraform configuration.

`main.tf`

```hcl
variable "access_key" {
  description = "AWS Access Key"
}

variable "secret_key" {
  description = "AWS Secret Key"
}

variable "region" {
  description = "AWS Region"
  default     = "us-east-1"
}

variable "ami" {
  description = "Server Image ID"
}

variable "subnet_id" {
  description = "Server Subnet ID"
}

variable "identity" {
  description = "Server Name"
}

variable "vpc_security_group_ids" {
  description = "Server Security Group ID(s)"
  type        = list(any)
}

provider "aws" {
  access_key = var.access_key
  secret_key = var.secret_key
  region     = var.region
}

module "server" {
  source = "./server"
  ami                    = var.ami
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  identity               = var.identity
}

output "public_ip" {
  value = module.server.public_ip
}

output "public_dns" {
  value = module.server.public_dns
}
```

Uncomment out the variables within the `terraform.tfvars` file.

## Task 3: Create Terraform Module

Create a directory called `server` and add the following files: `server.tf`, `variables.tf` and `outputs.tf`

`server.tf`

```hcl
resource "aws_instance" "web" {
  ami                    = var.ami
  instance_type          = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids

  key_name = var.key_name

  tags = {
    "Name"        = var.identity
    "Environment" = var.environment
    "createdby"   = "terraform"
  }
}
```

`variables.tf`

```hcl
variable "ami" {
  description = "amazon server image"
  default     = ""
}

variable "subnet_id" {
  description = "Subnet Id"
}

variable "vpc_security_group_ids" {
  description = "Security Groups"
  type        = list(any)
}

variable "key_name" {
  description = "SSH Key"
  default     = ""
}

variable "identity" {
  description = "Server Name"
}

variable "environment" {
  description = "Deployment Environment"
  default     = "development"
}
```

`outputs.tf`

```hcl
output "public_ip" {
  value = aws_instance.web.*.public_ip
}

output "public_dns" {
  value = aws_instance.web.*.public_dns
}
```

This will serve as a local Terraform module and the directory layout should look as follows:

```bash
├── main.tf
├── server
│   ├── server.tf
│   ├── outputs.tf
│   └── variables.tf
├── terraform.tfvars
```

## Task 4: Format and Validate Terraform Configuration

Initialize, Format and Validate your terraform configuration by executing the following from the `/workstation/terraform` directory in the code terminal.

```bash
cd /workstation/terraform
terraform init
terraform fmt -recursive
terraform validate
```

## Task 5: Deploy Infrastructure

Deploy the infrastructure via a `terraform plan` and `terraform apply`

```bash
terraform plan
terraform apply
```

```bash
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
```

```bash
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

Outputs:

public_dns = [
  "ec2-54-236-37-51.compute-1.amazonaws.com",
]
public_ip = [
  "54.236.37.51",
]
```
