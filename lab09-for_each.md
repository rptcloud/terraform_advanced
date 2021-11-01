# Lab: For-Each

Duration: 15 minutes

So far, we've already used arguments to configure your resources. These arguments are used by the provider to specify things like the AMI to use, and the type of instance to provision. Terraform also supports a number of _Meta-Arguments_, which changes the way Terraform configures the resources. For instance, it's not uncommon to provision multiple copies of the same resource. We can do that with the _count_ argument.

The count argument does however have a few limitations in that it is entirely dependent on the count index which can be shown by performing a `terraform state list`.

A more mature approach to create multiple instances while keeping code DRY is to leverage Terraform's `for-each`.

- Task 1: Change the number of AWS instances with `count`
- Task 2: Look at the number of AWS instances with `terraform state list`
- Task 3: Decrease the Count and determine which instance will be destroyed.
- Task 4: Refactor code to use Terraform `for-each`
- Task 5: Look at the number of AWS instances with `terraform state list`
- Task 6: Update the output variables to pull IP and DNS addresses.
- Task 7: Update the server variables to determine which instance will be destroyed.

## Task 1: Change the number of AWS instances with `count`

Update the root `main.tf` to utilize the `count` paramater on the both the server module and corresponding output blocks.  Notice the count is set to `2`.

```hcl
module "server" {
  count                  = 2
  source                 = "./server"
  ami                    = ""
  server_os              = var.server_os
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  identity               = var.identity
}

output "public_ip" {
  value = module.server.*.public_ip
}

output "public_dns" {
  value = module.server.*.public_dns
}
```

## Task 2: Look at the number of servers with `terraform state list`

Look at the server state reference before applying the `count` change:

```bash
terraform state list

module.server.aws_instance.web
```

Run a `terraform plan` to see the proposed changes:

```bash
terraform validate
terraform plan

Plan: 2 to add, 0 to change, 1 to destroy.
```

This may come as a suprise that the existing server needs to be destroyed when a new server is added using `count`.  This is because of the way resources are indexed when using meta-arguments.  Notice how the reference changes from `module.server.aws_instance.web` to `module.server[0].aws_instance.web` 

```
# module.server.aws_instance.web will be destroyed
# module.server[0].aws_instance.web will be created
# module.server[1].aws_instance.web will be created
```

Perform a `terraform apply` followed by a `terraform state list` to view how the servers are accounted for in Terraform's state.

```bash
terraform apply
```

```bash
terraform state list

module.server[0].aws_instance.web
module.server[1].aws_instance.web
```

## Task 3: Decrease the Count and determine which instance will be destroyed.

Update the count from `2` to `1`

```hcl
module "server" {
  count                  = 1
  source                 = "./server"
  ami                    = ""
  server_os              = var.server_os
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  identity               = var.identity
}
```

Run a `terraform apply` followed by a `terraform state list` to view how the servers are accounted for in Terraform's State.

```bash
terraform apply
```

```
terraform state list

module.server[0].aws_instance.web
```

You will see that when using the `count` parameter you have very limited control as to which server Terraform will destroy. It will always default to destroying the server with the highest index count.

## Task 4: Refactor code to use Terraform `for-each`

Refactor `main.tf` to make use of the `for-each` command rather then the count command. Replace the existing code in your `main.tf` with the following and comment out the `output` blocks for now.

```hcl
locals {
  servers = {
    server-iis = {
      server_os              = "windows_2019"
      identity               = "$var.identity-windows"
      subnet_id              = var.subnet_id
      vpc_security_group_ids = var.vpc_security_group_ids
    },
    server-apache = {
      server_os              = "ubuntu_20_04"
      identity               = "$var.identity-ubuntu"
      subnet_id              = var.subnet_id
      vpc_security_group_ids = var.vpc_security_group_ids
    }
  }
}

module "server" {
  source                 = "./server"
  for_each               = local.servers
  server_os              = each.value.server_os
  identity               = each.value.identity
  subnet_id              = each.value.subnet_id
  vpc_security_group_ids = each.value.vpc_security_group_ids
}

/*
output "public_ip" {
  value = module.server.*.public_ip
}

output "public_dns" {
  value = module.server.*.public_dns
}
*/
```

If you run `terraform apply` now, you'll notice that this code will destroy the previous resource and create two new servers based on the attributes defined inside the `servers` variable, which is defined as a map of our servers.

### Task 5: Look at the number of AWS instances with `terraform state list`

```bash
terraform state list

module.server["server-apache"].aws_instance.web
module.server["server-iis"].aws_instance.web
```

Since we used _for-each_ to the aws_instance.web resource, it now refers to multiple resources with key references from the `servers` variable.

### Task 6: Update the output variables to pull IP and DNS addresses.

When using Terraform's `for-each` our output blocks need to be updated to utilize `for` to loop through the server names. This differs from using `count` which utilized the Terraform splat operator `*`. Uncomment and update the output block of your `main.tf`.

```hcl
output "public_ip" {
  description = "Public IP of the Servers"
  value = { for p in sort(keys(local.servers)) : p => module.server[p].public_ip }
}

output "public_dns" {
  description = "Public DNS names of the Servers"
  value = { for p in sort(keys(local.servers)) : p => module.server[p].public_dns }
}
```

Format, validate and apply your configuration to now see the format of the Outputs.

```
terraform fmt
terraform validate
terraform apply
```

```bash
public_dns = {
  "server-apache" = [
    "ec2-3-238-102-62.compute-1.amazonaws.com",
  ]
  "server-iis" = [
    "ec2-3-238-152-59.compute-1.amazonaws.com",
  ]
}
public_ip = {
  "server-apache" = [
    "3.238.102.62",
  ]
  "server-iis" = [
    "3.238.152.59",
  ]
}
```

## Task 7: Update the server variables to determine which instance will be destroyed.

Update the `servers` local variable to remove the `server-iis` instance by removing the following block:

```hcl
    server-iis = {
      ami                    = "ami-07f5c641c23596eb9"
      instance_type          = "t2.micro",
      environment            = "dev"
      subnet_id              = "subnet-031bf0c9a309fcd8d"
      vpc_security_group_ids = ["sg-01380b40dc19ad166"]
    },
```

If you run `terraform apply` now, you'll notice that this code will destroy the `server-iis`, allowing us to target a specific instance that needs to be updated/removed.
