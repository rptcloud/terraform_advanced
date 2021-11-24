# Lab: Dynamic Blocks

Duration: 20 minutes

A dynamic block acts much like a for expression, but produces nested blocks instead of a complex typed value. It iterates over a given complex value, and generates a nested block for each element of that complex value. You can dynamically construct repeatable nested blocks using a special dynamic block type, which is supported inside resource, data, provider, and provisioner blocks.

- Task 1: Create a Security Group Resource with Terraform
- Task 2: Look at the state without a dynamic block
- Task 3: Convert Security Group to use dynamic block
- Task 4: Look at the state with a dynamic block
- Task 5: Use a dynamic block with Terraform map
- Task 6: Look at the state with a dynamic block using Terraform map

## Task 1: Create a Security Group Resource with Terraform

Create a new file in the root directory `/workstation/terraform` called `securitygroups.tf` with the following configuration.

**NOTE: replace the `###` `name = "core-sg-###"` with your initials.**

```hcl
resource "aws_security_group" "main" {
  name = "core-sg-###"
  # vpc_id = data.aws_vpc.main.id

  ingress {
    description = "Port 443"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    description = "Port 80"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

## Task 2: Look at the state without a dynamic block

Run a `terraform apply` followed by a `terraform state list` to view how the security groups are accounted for in Terraform's State.

```bash
terraform apply
```

```bash
terraform state show aws_security_group.main

# aws_security_group.main:
resource "aws_security_group" "main" {
    arn                    = "arn:aws:ec2:us-east-1:508140242758:security-group/sg-0faedaecacc2fb097"
    description            = "Managed by Terraform"
    egress                 = []
    id                     = "sg-0faedaecacc2fb097"
    ingress                = [
        {
            cidr_blocks      = [
                "0.0.0.0/0",
            ]
            description      = "Port 443"
            from_port        = 443
            ipv6_cidr_blocks = []
            prefix_list_ids  = []
            protocol         = "tcp"
            security_groups  = []
            self             = false
            to_port          = 443
        },
        {
            cidr_blocks      = [
                "0.0.0.0/0",
            ]
            description      = "Port 80"
            from_port        = 80
            ipv6_cidr_blocks = []
            prefix_list_ids  = []
            protocol         = "tcp"
            security_groups  = []
            self             = false
            to_port          = 80
        },
    ]
    name                   = "core-sg"
    owner_id               = "508140242758"
    revoke_rules_on_delete = false
    tags_all               = {}
    vpc_id                 = "vpc-04bd3a3beea4a12dd"
}
```

## Task 3: Convert Security Group to use dynamic block

Replace the contents of `securitygroups.tf` with the following configuration:

```hcl
locals {
  ingress_rules = [{
      port        = 443
      description = "Port 443"
    },
    {
      port        = 80
      description = "Port 80"
    }
  ]
}

resource "aws_security_group" "main" {
  name   = "core-sg"

  dynamic "ingress" {
    for_each = local.ingress_rules

    content {
      description = ingress.value.description
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```

## Task 4: Look at the state with a dynamic block

Run a `terraform apply` followed by a `terraform state list` to view how the securty groups are accounted for in Terraform's State.

```bash
terraform apply
```

```bash
terraform state show aws_security_group.main
# aws_security_group.main:
resource "aws_security_group" "main" {
    arn                    = "arn:aws:ec2:us-east-1:508140242758:security-group/sg-0aa7b9032c9e531d1"
    description            = "Managed by Terraform"
    egress                 = []
    id                     = "sg-0aa7b9032c9e531d1"
    ingress                = [
        {
            cidr_blocks      = [
                "0.0.0.0/0",
            ]
            description      = "Port 443"
            from_port        = 443
            ipv6_cidr_blocks = []
            prefix_list_ids  = []
            protocol         = "tcp"
            security_groups  = []
            self             = false
            to_port          = 443
        },
        {
            cidr_blocks      = [
                "0.0.0.0/0",
            ]
            description      = "Port 80"
            from_port        = 80
            ipv6_cidr_blocks = []
            prefix_list_ids  = []
            protocol         = "tcp"
            security_groups  = []
            self             = false
            to_port          = 80
        },
    ]
    name                   = "core-sg"
    owner_id               = "508140242758"
    revoke_rules_on_delete = false
    tags_all               = {}
    vpc_id                 = "vpc-04bd3a3beea4a12dd"
}
```

## Task 5: Use a dynamic block with Terraform map

Replace the contents of `securitygroups.tf` with the following configuration:

```hcl
variable "web_ingress" {
  type = map(object(
    {
      description = string
      port        = number
      protocol    = string
      cidr_blocks = list(string)
    }
  ))
  default = {
    "80" = {
      description = "Port 80"
      port        = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
    "443" = {
      description = "Port 443"
      port        = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}

resource "aws_security_group" "main" {
  name = "core-sg"

  dynamic "ingress" {
    for_each = var.web_ingress
    content {
      description = ingress.value.description
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
    }
  }
}
```

## Task 6: Look at the state with a dynamic block using Terraform map

Run a `terraform apply` followed by a `terraform state list` to view how the security groups are accounted for in Terraform's State.

```bash
terraform apply
```

```bash
terraform state show aws_security_group.main
# aws_security_group.main:
resource "aws_security_group" "main" {
    arn                    = "arn:aws:ec2:us-east-1:508140242758:security-group/sg-091494f2a09f6eed8"
    description            = "Managed by Terraform"
    egress                 = []
    id                     = "sg-091494f2a09f6eed8"
    ingress                = [
        {
            cidr_blocks      = [
                "0.0.0.0/0",
            ]
            description      = "Port 443"
            from_port        = 443
            ipv6_cidr_blocks = []
            prefix_list_ids  = []
            protocol         = "tcp"
            security_groups  = []
            self             = false
            to_port          = 443
        },
        {
            cidr_blocks      = [
                "0.0.0.0/0",
            ]
            description      = "Port 80"
            from_port        = 80
            ipv6_cidr_blocks = []
            prefix_list_ids  = []
            protocol         = "tcp"
            security_groups  = []
            self             = false
            to_port          = 80
        },
    ]
    name                   = "core-sg"
    owner_id               = "508140242758"
    revoke_rules_on_delete = false
    tags_all               = {}
    vpc_id                 = "vpc-04bd3a3beea4a12dd"
}
```

## Best Practices

Overuse of dynamic blocks can make configuration hard to read and maintain, so it is recommend to use them only when you need to hide details in order to build a clean user interface for a re-usable module. Always write nested blocks out literally where possible.
