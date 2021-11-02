# Lab: Terraform Policy as Code - Sentinel Policy Use

## Expected Outcome

In this challenge, you will see how you can apply policies using Sentinel Policy-as-Code.

## View Policies

Within Terraform Cloud/Enterprise, click on your organization -> Organization Settings

<https://app.terraform.io/app/YOUR_ORG_NAME/settings/policies>

![](img/sentinel-policy-add.png)

## Create Policy Set

First we need a place to store our policies, namely a Policy Set.

On the left menu, click the "Policy set" tab.

Click "Connect a new policy set"

![](img/sentinel-policyset-add-new.png)

As Sentinel Policies are code they are typically saved to Version Control, so we can select the code repository in our VCS providers in which our Sentinel policies reside.  We are going to add a VCS connection later, so we will select `No VCS Connection` at this time. 

Create the following policy:

![](img/sentinel-policyset-add-new-form.png)

Create the following policy:

__Name:__ GlobalWorkspacePolicies

__Description:__ Global policies.

__Scope of Policies:__ Select -> "Policies enforced on all workspaces"

Click "Connect policy set"

## Create Policy

Now lets create a Policy to enforce governance.

Under `Policies` - click "Create new policy"

![](img/sentinel-policy-add-new.png)

Create the following policy:

__Policy Name:__ RequireMandatoryTags

__Description:__ Policy Enforcing Mandatory Tags and Allowed Sizes

__Policy Enforcement:__ advisory (logging only)

__Policy Code:__

```hcl
   
# Imports mock data
import "tfplan/v2" as tfplan

# Get all AWS instances from all modules
ec2_instances = filter tfplan.resource_changes as _, rc {
    rc.type is "aws_instance" and
        (rc.change.actions contains "create" or rc.change.actions is ["update"])
}

# Mandatory Instance Tags
mandatory_tags = [
    "Name",
]

# Allowed Types
allowed_types = [
    "t2.micro",
    "t2.small",
    "t2.medium",
]

# Rule to enforce "Name" tag on all instances
mandatory_instance_tags = rule {
    all ec2_instances as _, instance {
        all mandatory_tags as mt {
            instance.change.after.tags contains mt
        }
    }
}

# Rule to restrict instance types
instance_type_allowed = rule {
    all ec2_instances as _, instance {
        instance.change.after.instance_type in allowed_types
    }
}

# Main rule that requires other rules to be true
main = rule {
    (instance_type_allowed and mandatory_instance_tags) else true
}
```

__Policy Sets__: Select the Policy Set we just created "GlobalWorkspacePolicies". (And be sure to click the add button to the right to `Add policy set`) and then click `Create Policy`

### Run a Plan

> Note: be sure to discard any existing plans.

Navigate to your "webserver-aws-dev" workspace and queue a plan.

### Review the Plan

Will see the plan was successful but there was a policy failure, however the option to Apply is still available. Why is that?

![](img/sentinel-advisory.png)

**Discard the plan.**

### Update the Policy

Navigate back the policy definition (/settings/policies)

Update the Policy Enforcement to be `hard-mandatory`.

### Run a Plan

Queue a plan for the workspace.

### Review the Plan

This time the the run fails due to the hard enforcement.

![](img/tfe-policy-fail.png)

## Advanced areas to explore

1. Write another Sentinel Policy restricting availablity zones to launch EC2 instances.

## Resources

- [Policy Examples](https://github.com/hashicorp/terraform-guides/tree/master/governance/second-generation)
- [Sentinel Language Spec](https://docs.hashicorp.com/sentinel/language/spec)
