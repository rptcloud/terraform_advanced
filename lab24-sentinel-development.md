# Lab: Terraform Enterprise - Sentinel Development

## Create Folder Structure

Create and change directory into a folder on your local machine.

For example: `cd ~/TerraformWorkshop/sentinel-development/`.

## Create Sentinel File

Create a new file `restrict-vm-disk-size.sentinel`.

Add imports:

```hcl
##### Imports #####

import "tfplan"
import "strings"
```

Add a list of allowed VM Sizes:

```hcl
##### Variables #####

max_disk_size = 100
```

Add a helper function that will find all resources of a specific type from all modules using the tfplan import:

```hcl
##### Functions #####

# Find all resources of a specific type from all modules using the tfplan import
find_resources_from_plan = func(type) {

  resources = {}

  # Iterate over all modules in the tfplan import
  for tfplan.module_paths as path {
    # Iterate over the named resources of desired type in the module
    for tfplan.module(path).resources[type] else {} as name, instances {
      # Iterate over resource instances
      for instances as index, r {

        # Get the address of the instance
        if length(path) == 0 {
          # root module
          address = type + "." + name + "[" + string(index) + "]"
        } else {
          # non-root module
          address = "module." + strings.join(path, ".module.") + "." +
                    type + "." + name + "[" + string(index) + "]"
        }

        # Add the instance to resources map, setting the key to the address
        resources[address] = r
      }
    }
  }

  return resources
}
```

Add another helper function to validate disk size

```hcl
# Validate that all VMware VMs have disk sizes beneath specified limit
validate_disk_size = func(disk_limit) {

  validated = true

  # Get all resources of specified type
  resource_instances = find_resources_from_plan("vsphere_virtual_machine")

  # Loop through the resource instances
  for resource_instances as address, r {

    # Skip resource instances that are being destroyed
    # to avoid unnecessary policy violations.
    # Used to be: if length(r.diff) == 0
    if r.destroy and not r.requires_new {
      print("Skipping resource", address, "that is being destroyed.")
      continue
    }

    # Initialize disk_count
    disk_count = -1

    # Get all disks for the VM
    for r.applied.disk as disk {

      # Increment disk_count for use in computed test
      disk_count += 1

      # Determine if the attribute is computed
      if r.diff["disk." + string(disk_count) + ".size"].computed else false is true {
        print("Virtual machine", address, "has disk", disk.label,
              "with attribute, disk.size, that is computed.")
        # If you want computed values to cause the policy to fail,
        # uncomment the next line.
        # validated = false
      } else {
        # Validate that each disk has valid disk size
        if int(disk.size) > disk_limit {
          print("Virtual machine", address, "has disk", disk.label, "with size",
                disk.size, "that is greater than the limit", disk_limit)
          validated = false
        }
      } // end computed check

    } // end disks

    print(address) # debug statement
  } // end resource instances

  return validated
}
```

Finally, add validation function and a main rule to tests is pass/fail:

```hcl
##### Rules #####

# Call the validation function
vm_disks_validated = validate_disk_size(max_disk_size)

# Main rule
main = rule {
  vm_disks_validated is true
}
```

> Note: If you get lost, see the solution folder for an example.

## Create a test folder structure

Create a the structure `test/restrict-vm-disk-size`.
Add a single file named `pass.json` in the new directory, with the following contents:

```json
{
  "mock": {
    "tfplan": "mock-tfplan-pass.sentinel"
  },
  "test": {
    "main": true
  }
}
```

Now create a file named `mock-tfplan-pass.sentinel` in the same directory.

## Download a mock

Back in TFE, navigate to the ` app-vm-dev` and find a previous successful run.

Click the "Download Sentinel Mocks" button.

![](img/sentinel-mocks-download.png)

Once downloaded, extract the contents and find the `tfplan` mock.

Copy the contents into the `mock-tfplan-success.sentinel` file you created in the last step.

## Run a test

In the `restrict-vm-disk-size.sentinel` file, locate the line:

```hcl
print(address) # debug statement
```

This is there to show how to get information out of a policy while testing.

Run this command: `sentinel test -verbose -run restrict-vm-disk-size.sentinel`

Your output should look something like this:

```sh
PASS - restrict-vm-disk-size.sentinel
  PASS - test/restrict-vm-disk-size/pass.json


    logs:
      vsphere_virtual_machine.vm[0]
    trace:
      TRUE - restrict-vm-disk-size.sentinel:104:1 - Rule "main"
```

Work with that debug statement to play around with the policy.

Success! You now have a valid sentinel policy!

## Write another test

Create a new file `test/restrict-vm-disk-size/fail.json`.

Create a new file `test/restrict-vm-disk-size/mock-fail-tfplan.sentintel`.

Update both files to represent a negative test.

## Resources

- [Sentinel Docs](https://docs.hashicorp.com/sentinel/)
- [Example VMware Policies](https://github.com/hashicorp/terraform-guides/tree/master/governance/second-generation/vmware)
- [More Policy Examples](https://github.com/hashicorp/tfe-policies-example)
