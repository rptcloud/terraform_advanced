# Lab: Terraform Cloud - Migrate to Remote State

In this lab you will utilize the [Terraform Cloud Remote Backend](https://app.terraform.io/signup?utm_source=banner&utm_campaign=intro_tf_cloud_remote).

Duration: 15 minutes

- Task 1: Update your Terraform configuration to use the remote backend
- Task 2: Check on your state file in Terraform Cloud
- Task 3: Manually delete local state file

## Task 1: Create remote backend

Within your `/workstation/terraform` update the `terraform.tf` and add the following information:

```shell
terraform {
  required_version = ">= 1.0.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
  backend "remote" {
    organization = "<ORGANIZATION NAME>"

    workspaces {
      name = "webserver-aws-dev"
    }
  }
}
```

Replace the organization with the name of Terraform Cloud Team & Goverance free trial organization name created in the previous lab.

The workspace name is arbitrary, since Terraform Cloud creates workspaces on demand; if a workspace with this name doesn't yet exist, it will be automatically created the next time you run terraform init for that configuration.

Run `terraform init`.

```shell
cd /workstation/terraform
terraform init
```

```bash
Initializing modules...

Initializing the backend...
Do you want to copy existing state to the new backend?
  Pre-existing state was found while migrating the previous "local" backend to the
  newly configured "remote" backend. No existing state was found in the newly
  configured "remote" backend. Do you want to copy this state to the new "remote"
  backend? Enter "yes" to copy and "no" to start with an empty state.

  Enter a value: yes
```

## Task 3: Check on your state file in Terraform Cloud

Now that you've migrated your state file, let's view it in Terraform Cloud to verify that everything worked correctly

- If you aren't already, log in to Terraform Cloud in your browser
- Navigate to your new workspace
- In your workspace, click on the `States` tab
- You should see something along the lines of this in the `States` tab. This indicates that your state file was successfully migrated, and from now on, you can check here after new applies for new versions of your state file.
  - ![remote state file](./img/remoteState.png)

## Task 3: Manually delete local state file

After confirming that TFC is now managing your state file the local `terraform.tfstate` can be deleted.

> Note: Only perform this step if you have validated your state file has been successful migrated to Terraform Cloud.

```
rm terraform.tfstate
```

## At this point in time, your state file is migrated, but there is still work to be done. We will need to do the next few labs before we can successfully run a new apply.
