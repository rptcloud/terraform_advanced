# Lab: Sentinel Policy-as-Code Testing
Sentinel is the Policy-as-Code product from HashiCorp that automatically enforces logic-based policy decisions across all HashiCorp Enterprise products.

It allows users to implement policy-as-code in a similar way to how Terraform implements infrastructure-as-code. The Sentinel Command Line Interface (CLI) allows you to apply and test Sentinel policies, including those that use mocks generated from Terraform Cloud and Terraform Enterprise plans.

- Task 1: Setup Workspace in Terraform Cloud
- Task 2: Download Sentinel CLI
- Task 3: Create Sentinel Policy
- Task 4: Testing Sentinel Policy
- Task 5: Enforce Sentinel Policy

## Task 1: Setup Workspace in Terraform Cloud 
You should already have a Terraform Cloud account with Team and Governance capabilities, if it has been less 30 days since you activated the trial on your Terraform Cloud account you are good to go. If you do not have a Terraform Cloud account already, please create an account.

## Task 2: Download Sentinel CLI

The Sentinel CLI is a command-line interface for local development and testing. For the getting started guide, we'll use the CLI to learn how to write policies for Sentinel-enabled applications. The Sentinel CLI is distributed as a binary package for all supported platforms and architectures.

```bash
curl -sfLo sentinel.zip https://releases.hashicorp.com/sentinel/0.18.4/sentinel_0.18.4_linux_amd64.zip
unzip -qq sentinel.zip 
which sentinel
sudo mv sentinel /usr/local/bin/sentinel
sentinel version
```

## Task 3: Create Sentinel Policy
Now that we have a working Terraform configuration and workspace, lets write a policy to set guardrails around one of type of resources created by our config. We want to enforce 2 organization requirements for EC2:

- EC2 instances must have a Name tag.
- EC2 instances must be of type t2.micro, t2.small, or t2.medium.

We will leverage the [HashiCorp Learn Tutorials](https://learn.hashicorp.com/collections/terraform/policy) for building out our policies.

Browse to the [Write a Sentinel Policy for a Terraform Deployment](https://learn.hashicorp.com/tutorials/terraform/sentinel-policy?in=terraform/policy)

Clone down the example repository for writing sentinel policy

```bash
git clone https://github.com/hashicorp/learn-sentinel-write-policy
cd learn-sentinel-write-policy
```

Run your policy in the Sentinel CLI

```bash
sentinel apply restrict-aws-instances-type-and-tag.sentinel
```

## Task 4: Testing Sentinel Policy
Testing Sentinel policies with the built-in testing suite ensures that you account for all possible behaviors in your policy, and that Sentinel operates as expected when Terraform Cloud applies these policies within your organization.

Browse to the [Test Sentinel Policies](https://learn.hashicorp.com/tutorials/terraform/sentinel-testing?in=terraform/policy)

### Creating a Passing Test
Copy your known passing mock data to a new file with `pass` in the filename.

```bash
cp mock-tfplan-v2.sentinel mock-tfplan-pass-v2.sentinel
```

Create the required testing folder structure for executing Sentinel Policy tests

```bash
mkdir -p test/restrict-aws-instances-type-and-tag
cd test/restrict-aws-instances-type-and-tag
```

Create a new file named `pass.hcl`, then add the following configuration to the file.

`pass.hcl`
```sentinel
mock "tfplan/v2" {
  module {
    source = "../../mock-tfplan-pass-v2.sentinel"
  }
}

test {
    rules = {
        main = true
    }
}
```

### Creating a Failing Test
Copy your known passing mock data to a new file with `fail` in the filename.

```bash
cd ../..
cp mock-tfplan-v2.sentinel mock-tfplan-fail-v2.sentinel
```

Edit the `mock-tfplan-fail-v2.sentinel` and replace the items in the `resource_changes.aws_instance.ubuntu.change.after` block

```text
instance_type to t2.large
tags from Name to Number
```

Change back to testing folder structure and creat a new file named `fail.hcl`

```bash
cd test/restrict-aws-instances-type-and-tag
```

`fail.hcl`
```sentinel
mock "tfplan/v2" {
  module {
    source = "../../mock-tfplan-fail-v2.sentinel"
  }
}

test {
    rules = {
        main = false
    }
}
```

### Testing Pass and Fail Conditions

Change into your root learn-sentinel-policies directory and run your sentinel tests.

```bash
cd ../..
```

```bash
sentinel test restrict-aws-instances-type-and-tag.sentinel
```

Run the tests in verbose mode.

```bash
sentinel test -verbose restrict-aws-instances-type-and-tag.sentinel
```

## Resources
- [Enforce Policy with Sentinel - Learn Tracks](https://learn.hashicorp.com/collections/terraform/policy)
- [Experiment in the Sentinel Playground](https://play.sentinelproject.io/)