# Lab: Local Variables

Duration: 15 minutes

A local value assigns a name to an expression, so you can use it multiple times within a module without repeating it. The expressions in local values are not limited to literal constants; they can also reference other values in the module in order to transform or combine them, including variables, resource attributes, or other local values.

- Task 1: Create local variables in a configuration block
- Task 2: Interpolate local variables
- Task 3: Using locals with variable expressions
- Task 4: Using locals with terraform expressions and operators

## Task 1: Create local variables in a configuration block

Add local variables to your `server.tf` module directory:

```hcl
locals {
  service_name = "Automation"
  owner        = "Cloud Team"
  createdby    = "terraform"
}
```

## Task 2: Interpolate local variables into your existing code

Update the `aws_instance` block inside your `server.tf` to add new tags to all instances using local variable interpolation.

```hcl
...

 tags = {
    "Name"        = var.identity
    "Environment" = var.environment
    "createdby"   = local.createdby
    "Service"     = local.service_name
    "Owner"       = local.owner
  }

...
```

After making these changes, rerun `terraform plan`. You should see that there will be some tagging updates to your server instances. Execute a `terraform apply` to make these changes.

## Task 3: Using locals with variable expressions

Expressions in local values are not limited to literal constants; they can also reference other values in the module in order to transform or combine them, including variables, resource attributes, or other local values.

Add another local variable block to your `server.tf` module configuration which references the local variables set in the previous portion of the lab.

```hcl
locals {
  # Common tags to be assigned to all resources
  common_tags = {
    Name        = var.identity
    Environment = var.environment
    createdby = local.createdby
    Service = local.service_name
    Owner   = local.owner
  }
}
```

Update the `aws_instance` tags block inside your `server.tf` to reference the `local.common_tags` variable.

```hcl
resource "aws_instance" "web" {
  ami                    = var.ami
  instance_type          = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids

  tags = local.common_tags
}
```

After making these changes, rerun `terraform plan`. You should see that there are no changes to apply, which is correct, since the variables contain the same values we had previously hard-coded, but now that are

```text
...

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```

## Task 4: Using locals with terraform expressions and operators

Locals are commonly used to transform or combine values using expressions supported within HCL. This can help make the code based within a given resource block easier to read. We use locals within our module to be able to allow module consumers the ability to specify a server operating system or ami image id for which to build our server.

Add the following locals block to your `server.tf` module, which utilizes a terraform conditional operator to determine the image id for a given server. This allows a consumer of the module to specify their own AMI or to have terraform select the appropriate AMI based on server operating system. This logic is handled within the terraform local variable and then reference within the `aws_instance` resource block.

Validate that your `data.tf` file within your server module directory has the following data blocks defined (this was completed in previous labs):

`data.tf`

```hcl
data "aws_ami" "windows_2019" {
  most_recent = true
  filter {
    name   = "name"
    values = ["Windows_Server-2019-English-Full-Base-*"]
  }
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
  owners = ["801119661308"]
}

data "aws_ami" "ubuntu_20_04" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}

data "aws_ami" "ubuntu_18_04" {
  most_recent = true

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }

  owners = ["099720109477"]
}
```

Add the following code to your `server.tf` file to allow your module consumers the ability to specify their own AMI image id, or to use one of the default server operating systems. Notice the `ami` local value is determined using a conditional operator. This logic determines if an `ami` value was provided or needs to be looked up using a data block.

`server.tf`

```hcl
locals {
  server_os = {
    "ubuntu_18_04" = data.aws_ami.ubuntu_18_04.image_id
    "ubuntu_20_04" = data.aws_ami.ubuntu_20_04.image_id
    "windows_2019" = data.aws_ami.windows_2019.image_id
  }

  ami = (var.ami != "" ? var.ami : lookup(local.server_os, var.server_os))

}
```

Now we can update the `aws_instance` resource block within your `server.tf` to call the `local.ami` value rather then building the logic directly inside the resource block. This makes our code easier to read.

`server.tf`

```hcl
...

resource "aws_instance" "web" {
  ami                    = local.ami
  instance_type          = "t2.micro"

...
```

After making these changes, rerun `terraform plan`. You should see that there are no changes to apply since the code is using the user specified ami.

```text
...

No changes. Infrastructure is up-to-date.

This means that Terraform did not detect any differences between your
configuration and real physical resources that exist. As a result, no
actions need to be performed.
```

Local values can be helpful to avoid repeating the same values or expressions multiple times in a configuration, but if overused they can also make a configuration hard to read by future maintainers by hiding the actual values used.

Use local values only in moderation, in situations where a single value or result is used in many places and that value is likely to be changed in future. The ability to easily change the value in a central place is the key advantage of local values.
