# Lab: Terraform Enterprise - Private Module Registry

In this lab, we will be taking a look at a few of the publicly available modules, adding one to our private module registry, and then deploying it into our code.

Duration: 20 minutes

- Task 1: Find the Public Module Registry
- Task 2: Add public module to your private module registry
- Task 3: Reference the new module in your Terraform configuration

## Task 1: Find the Public Module Registry

- Log into Terraform Cloud if you are not already
- Click on the link in the top ribbon called `Registry`.
- Click on the link in the top right hand corner called `Find public modules`.

## Task 2: Add public module to your private module registry

Here, you should be able to go through the public module registry and search for something that is relevant to your use case. Once the module is added to your private module registry from the public one, you can also begin to make changes to it to better suit your needs.

In order to add a public module to your private module registry:

- In the search bar that says `Find Public Modules`, let's type in `S3` and see what is returned
- The first result should be a module called `terraform-aws-modules/s3-bucket`. Let's click on that one.
- Once here, take a minute to go through the module page to familiarize yourself with what the module does, as well as what inputs and outputs we can use with it. Pay special attention to the Inputs section, as we will need to construct a few lines of code that will allow us to call this module.
- Once you have gone through this page, you can click on the purple box in the top right corner that says `+ Add to Terraform Cloud`. This will then prompt you to confirm your choice to add this to you organization. Click `Add to organization`.

## Task 3: Reference the new module in your Terraform configuration

Now that you've added this module to your private module registry, we can work with it.

- Navigate back to your private module registry.  You can do this by clicking back on the `Registry` link in the top ribbon.
- In the search bar, you should be able to type `s3`. This should return the module that you just added to your private registry. Let's click into that.
- Now that we have our module, let's go ahead and add it to our code.

In our `main.tf` file, we can add the following block to our code. This is simply referenced from the configuration details in the module page:

```hcl
module "s3-bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"
  version = "2.7.0"
  # insert required variables here
}
```

- Update the copied code block with the required variables.  Let's modify our code block to include the following information:

```hcl
variable bucket {}
variable acl {}

module "s3-bucket" {
  source  = "terraform-aws-modules/s3-bucket/aws"
  version = "2.7.0"

  bucket = var.bucket
  acl    = var.acl

  versioning = {
    enabled = true
  }
}
```

- Lastly, we'll have to give values to our variables in Terraform Cloud.

Navigate to your workspace where we have previously set up our variables. We'll have to declare two new variables in order for our module to work properly.

Let's do the following

- Create a variable called `bucket`. Give it a value with your initials and append it with `-tf201-training-bucket`. For instance, `rpt-tf201-training-bucket`.
- Create a variable called `acl`. Give it a value equal to `private`.

At this point in time, we should be good to go ahead and re-run our Terraform configuration. Let's do that now and check for a successful run.

```bash
terraform init
terraform plan
terraform apply
```

```bash
Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```