# Lab: Data Resources and Conditions

Duration: 15 minutes

So far, we've already used arguments to configure your resources. These arguments are used by the provider to specify things like the AMI to use, and the type of instance to provision. Terraform also supports the ability to look up information through the use of `data` resources.

For instance, it's not uncommon to need to query the value of an AMI type within a regions for an EC2 instance. AMIs are specific to each region therefore it is often more vaulable to query for this value rather then specifying it in a variable. This is where the use of a `data` resource block within HCL is convienent. To expand on this process we will also introduce the Conditional Expression, allowing us to determine if we want to pull a Linux or Windows image.

- Task 1: Query the AMI Image for Ubuntu and Windows using a Terraform `data` resource.
- Task 2: Look at the returned value of the data resources using a `terraform state list` and `terraform show` command.
- Task 3: Leverage Terraform's interpolation to specify the AMI for our `aws_instance`
- Task 4: Use Terraform conditional operators to determine the image/operating system of our `aws_instance`
- Task 5: Change the OS type for the server updating the root module and adding a `server_os` variable.

## Task 1: Query the AMI Image for Ubuntu and Windows using a Terraform `data` resource.

Add a `data.tf` file into the `server` module directory with the following items.

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

### Task 2: Look at the returned value of the data resources using a `terraform state list` and `terraform show` command.

Run a `terraform init` followed by a `terraform apply` to query the data items specified and view those items by issuing a `terraform state list`.

```bash
terraform init
terraform apply
```

> Note: If this `apply` resulted in a a `No Changes` then the state list command below will not show any entries, so you can move to Task 3.

```bash
terraform state list
module.server.data.aws_ami.ubuntu_18_04
module.server.data.aws_ami.ubuntu_20_04
module.server.data.aws_ami.windows_2019
```

```bash
terraform state show module.server.data.aws_ami.ubuntu_18_04
```

```bash
# module.server.data.aws_ami.ubuntu_18_04:
data "aws_ami" "ubuntu" {
    architecture          = "x86_64"
    arn                   = "arn:aws:ec2:us-east-1::image/ami-02fe94dee086c0c37"
    block_device_mappings = [
        {
            device_name  = "/dev/sda1"
            ebs          = {
                "delete_on_termination" = "true"
                "encrypted"             = "false"
                "iops"                  = "0"
                "snapshot_id"           = "snap-074c9b6e7aeb0e066"
                "throughput"            = "0"
                "volume_size"           = "8"
                "volume_type"           = "gp2"
            }
            no_device    = ""
            virtual_name = ""
        },
#...
```

### Task 3: Leverage Terraform's interpolation to specify the AMI for our `aws_instance`

Update the `server.tf` in the `server` module directory changing out `ami` attribute in the `aws_instance.web` resource block.

```hcl
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu_18_04.image_id
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

Run a `terraform apply` to build out the EC2 instance using the AMI that was looked up from the data lookup and validate with a `terraform state show module.server.aws_instance.web` to see that the server was build using the AMI from the data interpolation.

```bash
terraform state show module.server.aws_instance.web

# module.server.aws_instance.web:
resource "aws_instance" "web" {
    ami                          = "ami-02fe94dee086c0c37"
```

### Task 4: Use Terraform conditional operators to determine the image operating system for our `aws_instance`

A conditional expression uses the value of a bool expression to select one of two values. The syntax of a conditional expression is as follows:

```hcl
condition ? true_val : false_val
```

We create and utilize a variable to determine which AMI will be used via a conditional expression. Update the `variables.tf` of the `server` module directly to add a new variable for `server_os`

```hcl
variable server_os {
    type = string
    description = "Server Operating System"
    default = "ubuntu_20_04"
}
```

Update the `server.tf` of the `server` module to include a conditional operator for selecting an AMI based on the value of the `server_os` variable.

```
resource "aws_instance" "web" {
  ami                    = (var.server_os == "ubuntu_20_04") ? data.aws_ami.ubuntu_20_04.image_id : data.aws_ami.windows_2019.image_id
  instance_type          = "t2.micro"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids

   tags = {
    "Name"        = var.identity
    "Environment" = var.environment
    "createdby"   = "terraform"
  }
}
```

Run a `terraform apply` to validate that the new image is being pulled which may force the server to be replaced.

### Task 5: Change the OS type for the server updating the root module and adding a `server_os` variable.

Change the value of your `server_os` variable to now specify `windows_2019` and the conditional operator will query for the Windows AMI using the `data` interpolation.

This can be done within the variable definition of the module, or it can be done in the `main.tf` within the root module by adding the following to the configuration.

`main.tf` of root module (set `ami = ""` and add `server_os`)

```hcl
module "server" {
  source                 = "./server"
  ami                    = ""
  server_os              = "windows_2019"
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  identity               = var.identity
}
```

```bash
terraform validate
terraform apply
```

```bash
  # module.server.aws_instance.web must be replaced
-/+ resource "aws_instance" "web" {
      ~ ami                                  = "ami-083654bd07b5da81d" -> "ami-0416f96ae3d1a3f29" # forces replacement
```

You can now variablize the operating system of your server by creating the definintion within the `main.tf` of your root module and the variable decleration within TFC.  Also update the `module` block to reference this variable.

`variables.tf`

```hcl
variable "server_os" {
  type        = string
  description = "Server Operating System"
  default     = "ubuntu_20_04"
}

module "server" {
  source                 = "./server"
  ami                    = ""
  server_os              = var.server_os
  subnet_id              = var.subnet_id
  vpc_security_group_ids = var.vpc_security_group_ids
  identity               = var.identity
}
```

![Server OS Variable](./img/server_os.png)

```bash
terraform validate
terraform apply
```

This shouuld result in a zero change plan, as we have simply refactored the code to make use of variables but not change the variable value itself.

```bash
module.server.aws_instance.web: Refreshing state... [id=i-084964bc3f42e390e]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration
and found no differences, so no changes are needed.
```
