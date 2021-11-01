# Lab: Terraform Cloud Design Configuration

Duration: 20 minutes

This lab demonstrates how to consume modules in the private module registry.

- Task 1: Configure the module with the web UI
- Task 2: Launch the configuration designer
- Task 3: Configure variables

## Prerequisites

For this lab, we'll assume that you've installed [Terraform](https://www.terraform.io/downloads.html) and that you have a [GitHub](https://github.com) account.

## Task 1: Configure the module with the web UI

The Terraform Cloud module configuration designer supports a producer/consumer pattern where some teams create modules and other teams use them to create infrastructure. You'll use the configuration designer to generate code that can be copy-and-pasted into a new Terraform project.

## Task 2: Launch the configuration designer

Start by either clicking the "Open in Configuration Designer" button under the right hand code snippet, or go back to the organization dashboard and click the "+ Design Configuration" button.

![Module Design Configuration](./img/module-design-configuration.png "Module Design Configuration")

In either case, you'll see a screen with a list of modules. Click the "Add Module" button on the `server` module.

## Task 3: Configure variables

Click the green "Next" button to proceed to the configuration screen. You'll see a list of variables, a description of each, and an input field where you can type a value for the variable.

![Module Variables](./img/module-variables.png "Module Variables")

Type any name into the name field, such as "web".

Click the large green "Next" button on the top right.

You'll be taken to a screen where you can preview the generated code, or download it as a file (it will be named `main.tf`).

In a production scenario, you would save this file to a new or existing Terraform project, add it to a repository in your source code control system, and connect the repository to Terraform Cloud for provisioning.
