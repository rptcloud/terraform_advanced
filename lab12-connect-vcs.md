# Lab: Terraform Cloud - VCS Connection

In this challenge, you will connect TFC to your personal github account.

Duration: 15 minutes

- Task 1: Create the VCS Connection
- Task 2: Verify Connection

## Task 1: Create the VCS Connection

1. Login to github in one browser tab.
2. Login to TFC in another browser tab.
3. Within TFC, navigate to the settings page:

![](img/tfe-settings.png)

4. Click "Providers" link under `Version` tab:

![](img/tfe-settings-vcs.png)

5. Following the instructions on the documents page <https://www.terraform.io/docs/cloud/vcs/github.html>

> The process involves several back and forth changes and is documented well in the link.

## Task 2: Verify Connection

Navigate to [<TFC>](https://app.terraform.io/) and click "+ New Workspace".

Click on `Version Control Workflow Connection` in the "Choose your Workflow" section.  Connect to the Version Control Provider that you just created and Verify you can see repositories:

![](img/tfe-vcs-verify.png)

If you can see repositories then you are good :+1:.

We will use this connection in future labs.  You can cancel out of the process.
