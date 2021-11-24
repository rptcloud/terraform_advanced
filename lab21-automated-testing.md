# Lab: Automated Testing

We may want to test our infrastructure to ensure it is healthy and behaving how we want it to.

Duration: 15 minutes

- Task 1: Write a unit test for your code
- Task 2: Use Terratest to Deploy infrastructure
- Task 3: Validate infrastructure with Terratest
- Task 4: Undeploy

The only real way to test infrastructure code beyond static analysis is by deploying it to a real environment, whatever environment you happen to be using.

[Terratest](https://terratest.gruntwork.io) is a Go library that provides patterns and helper functions for testing infrastructure, with 1st-class support for Terraform, Packer, Docker, Kubernetes, AWS, GCP, and more.

## Task 1: Write a unit test for your code

First, ensure you are in the `/workstation/terraform/` directory in your workstation.

Install Go on your training workstation

```bash
 wget -c https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz -O - | sudo tar -xz -C /usr/local
export PATH=$PATH:/usr/local/go/bin
sudo apt install golang-go
```

```bash
go version
```

Inside of the `/workstation/terraform/` directory create a `testing_lab` folder add the following Terraform configuration files from [here](https://github.com/rptcloud/terraform_training/tree/master/lab_solutions_09). Make sure they follow that exact structure.

```shell
mkdir -p /workstation/terraform/testing_lab && cd $_
```

In the `/workstation/terraform/testing_lab/` directory, copy your `terraform.tfvars` file and it has the following contents:

```hcl
 access_key        = "(your AWS access key id)"
 secret_key        = "(your AWS secret access key)"
 ami               = "(your ami)"
 subnet_id         = "(your subnet id)"
 identity          = "terraform-training-(your animal)"
 region            = "(your region)"
 vpc_security_group_ids = ["(your sg)"]
```

If these values are commented out (they are by default), make sure you uncomment them as our Terraform configuration will be using them.  Also copy the `assets` folder containing your web application into the `testing_lab` directory.

Create a new folder within the `/workstation/terraform/testing_lab/server` folder called `test`. This will house your test for the server module.

In the `test` folder, create a file ending in server_test.go

`server_test.go`

```go
package test

import (
	"testing"
	"fmt"
	"net/http"
	"github.com/gruntwork-io/terratest/modules/shell"
	"github.com/gruntwork-io/terratest/modules/terraform"
	"github.com/stretchr/testify/assert"
)

func TestEnvironment(t *testing.T) {
	t.Parallel()

	// Configuring the Terraform Options that we use to pass into terraform. We have an environment variables map to declare env variables. We also
	// configure the options with default retryable errors to handle the most common retryable errors encountered in
	// terraform testing.
	terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
		// The path to where our Terraform code is located
		TerraformDir: "../../",
	})

	// defer is like a try finally, where at the end of this test, this line will always run. This line calls a Terraform destroy, which always gets called.
	defer terraform.Destroy(t, terraformOptions)

	// Run `terraform init` and `terraform apply`. The test fails if there are any errors
	terraform.InitAndApply(t, terraformOptions)

	server_dns := terraform.OutputList(t, terraformOptions, "public_dns")
	server_ip := terraform.OutputList(t, terraformOptions, "public_ip")
	
	//pings the server ips, will fail if they do not ping. The ping will wait for 60 seconds to ensure the ip is ready and can be pinged.
	
	for i := 0; i < len(server_ip); i++ {
		cmd := shell.Command{
			Command: "ping",
			Args:    []string{"-w", "180", "-c", "10", server_ip[i]},
		}
		shell.RunCommandAndGetOutput(t, cmd)
	}

	for i := 0; i < len(server_dns); i++ {
		//ensure that you can http get the servers and the response is 200
		resp, err := http.Get("http://" + server_dns[i])
		assert.Nil(t, err)
		defer resp.Body.Close()
		fmt.Print("HTTP request on " + server_dns[i] + " was ")
		fmt.Println(resp.StatusCode)
		assert.Equal(t, 200, resp.StatusCode)
	}

}
```

At the end of this task you should have a file layout similar to the following:

```shell
.
├── assets
│   ├── setup-web.sh
│   ├── webapp
│   └── webapp.service
├── main.tf
├── outputs.tf
├── server
│   ├── server.tf
│   └── test
│       └── server_test.go
├── terraform.tfvars
└── variables.tf
```

## Task 2:  Use Terratest to Deploy infrastructure
We will use Terratest to execute terraform to deploy our infrastructure into AWS.

```bash
cd /workstation/terraform/testing_lab/server/test
test_file="$(ls *test.go)"
go mod init "${test_file%.*}"
go mod tidy
go test -v $test_file
```
**Note: Go tests have a default timeout of 10 minutes. If your infrastructure takes longer than 10 minutes to create, you may want to add the optional `-timeout` flag when running your go test. For a timeout of 30 minutes, you would do: `go test -v -timeout 30m $test_file`**

If working correctly, the test should output something along the lines of:

```
TestEnvironment 2021-08-19T14:49:55Z logger.go:66: Destroy complete! Resources: 7 destroyed.
TestEnvironment 2021-08-19T14:49:55Z logger.go:66: 
--- PASS: TestEnvironment (133.33s)
PASS
ok      command-line-arguments  133.336s
student@terraform-training-chipmunk:/workstation/terraform/test > 
```


## Task 3: Validate infrastructure with Terratest

Terratest allows us to validate that the infrastructure works correctly in that environment by making HTTP requests, API calls, SSH connections, etc.

For a full list of every function Terratest provides, visit their documentation [here](https://pkg.go.dev/github.com/gruntwork-io/terratest)

While Terratest has many built-in functions, you can also use other Go packages in conjunction with Terratest. For instance, you can create a Terraform configuration that creates an EC2 instance with specific tags. In conjunction with the AWS package in Go, you can connect to AWS and use the AWS Go package's functions to ensure the EC2 exists and has the specified tags in your configuration file.

Finally, you can have your test fail if something is not as it should be. With the "assert" package in Go, you can ensure your outputs are as expected, causing the test to fail if they are not.



## Task 4: Undeploy
The final step of our test is to undeploy everything at the end. Terratest allows us to perform a terraform destroy at the end of the testing cycle. Take a look inside of your `server_test.go` file. You should be able to find the following lines:

```go
	// defer is like a try finally, where at the end of this test, this line will always run. This line calls a Terraform destroy, which always gets called.
	defer terraform.Destroy(t, terraformOptions)
```

In Go, defer is a statement that will tell your test to run this command last no matter what. Even if the test fails, or errors out somewhere in the code during runtime, this terraform.Destroy line will always run to ensure your test infrastructure doesn't become unmanaged by Terraform and difficult to find.
