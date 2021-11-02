# Lab: Terraform Policy as Code - Sentinel Policy

In this challenge, you will see how you can apply policies using Sentinel Policy-as-Code.

Duration: 20 minutes

- Task 1: The Sentinel CLI
- Task 2: Create a Sentinel Policy
- Task 3: Test Sentinel Policy

Sentinel allows customers to implement policy-as-code in the same way that Terraform implements infrastructure-as-code.

The Sentinel Command Line Interface (CLI) allows you to apply and test Sentinel policies including those that use mocks generated from Terraform Cloud and Terraform Enterprise plans.

## Task 1: The Sentinel CLI

The Sentinel Command Line Interface (CLI) allows you to apply and test Sentinel policies including those that use mocks generated from Terraform Cloud plans.

> Note: To install the latest version of the Sentinel CLI perform the following steps:

```shell
mkdir -p /workstation/terraform/sentinel && cd $_
curl -sfLo "sentinel.zip" "https://releases.hashicorp.com/sentinel/0.18.4/sentinel_0.18.4_linux_amd64.zip"
unzip -qq "sentinel.zip"
sudo mv "sentinel" "/usr/local/bin/sentinel"
sudo chmod +x "/usr/local/bin/sentinel"
rm -rf "sentinel.zip"
```

Let's start with some basic Sentinel commands, running them in the "Sentinel CLI" tab on the left.

Check the version of Sentinel running on your machine:

```shell
sentinel version
```

See the list of Sentinel CLI commands:

```shell
sentinel
```

Get help for the "sentinel apply" command:

```shell
sentinel apply -h
```

Please read the output to learn about the "apply" command.

Note that you can also use the "-help" and "--help" flags instead of "-h".

Get help for the "sentinel test" command:

```shell
sentinel test -h
```

## Task 2: Create a Sentinel Policy

The Sentinel "apply" command lets you evaluate Sentinel policies: https://docs.hashicorp.com/sentinel/commands/apply

The Sentinel "test" command lets you test a Sentinel policy against multiple test cases: https://docs.hashicorp.com/sentinel/commands/test

Both commands use Sentinel CLI configuration files: https://docs.hashicorp.com/sentinel/commands/config

```shell
cd /workstation/terraform/sentinel
touch require-even-number.sentinel
```

`require-even-number.sentinel`

```ruby
# A policy that requires an integer to be even

# A parameter that must be set to an integer.
param the_number default 1

# Print the number
print("The number is:", the_number)

# The main rule
main = rule {
  # Divide the integer by 2 and compare the remainder to 0
  the_number % 2 is 0
}
```

Apply the "require-even-number.sentinel" policy with the default value of 1 on the "Sentinel CLI" tab:

```shell
sentinel apply require-even-number.sentinel
```

This will fail since 1 is an odd number.

Apply the "require-even-number.sentinel" policy with a value of 2:

```shell
sentinel apply -param the_number=2 require-even-number.sentinel
```

This will pass since 2 is an even number.

Run that command again, but add -trace before the name of the policy to make Sentinel print more information:

```shell
sentinel apply -param the_number=2 -trace require-even-number.sentinel
```

Note that adding -trace causes the sentinel test command to print output from any of the policy's print statements even when a test case passes.

## Task 3: Test the Policy

Now, we will test the policy with the "sentinel test" command using test cases. For any policy, the test cases must be placed under the directory test/<policy> under the directory containing the policy where <policy> is the name of the policy without the ".sentinel" extension.

Create a `/test/require-even-number` folder and place both a pass and fail set of tests in the directory.
  
```shell
.
├── require-even-number.sentinel
└── test
    └── require-even-number
        ├── fail.hcl
        └── pass.hcl
```

`fail.hcl`

```ruby
param "the_number" {
     value = 7
}

test {
  rules = {
    main = false
  }
}
```

`pass.hcl`

```ruby
param "the_number" {
     value = 10
}

test {
  rules = {
    main = true
  }
}
```

Edit the "fail.hcl" test case for the "require-even-number.sentinel" policy on the "Test Cases" tab, replacing <an_odd_number> with any odd number. Save the "fail.hcl" file by clicking the disk icon above the file.

Edit the "pass.hcl" test case for the "require-even-number.sentinel" policy on the "Test Cases" tab, replacing <an_even_number> with any even number. Save the "pass.hcl" file by clicking the disk icon above the file.

Test the "require-even-number.sentinel" policy against both test cases on the "Sentinel CLI" tab:

```shell
sentinel test -run=number -verbose
```

```shell
Installing test modules for test/require-even-number/fail.hcl
Installing test modules for test/require-even-number/pass.hcl

PASS - require-even-number.sentinel
  PASS - test/require-even-number/fail.hcl


    logs:
      The number is: 7.000000
    trace:
      require-even-number.sentinel:10:1 - Rule "main"
        Description:
          The main rule

        Value:
          false
  PASS - test/require-even-number/pass.hcl


    logs:
      The number is: 10.000000
    trace:
      require-even-number.sentinel:10:1 - Rule "main"
        Description:
          The main rule

        Value:
          true
```

Both test cases should pass and show you the numbers that were used.

Note that when specifying a policy with the "-run" argument, you can match any portion of the policy name. Often, you will want to use something that only matches a single policy in the current directory.

Please read the output to learn about the "test" command.

The "sentinel apply" and "sentinel test" commands both evaluate a Sentinel policy, but the latter tests it against multiple test case files.
