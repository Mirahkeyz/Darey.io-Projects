# AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM- PART 3 (REFACTORING)

In two previous projects, we developed AWS Infrastructure code using Terraform and tried to run it from our local workstation. In this project,we will introduce some more advanced concepts and enhance the code.

We will explore alternative Terraform backends. A backend defines where Terraform stores its state data files - terraform.tfstate file.

Terraform uses persisted state data to keep track of the resources it manages. Most non-trivial Terraform configurations either integrate with Terraform Cloud or use a backend to store state remotely. This lets multiple people access the state data and work together on that collection of infrastructure resources.

# REFACTORING THE CODE USING MODULES
We will be refactoring our codes to use modules and move the terraform.tfstate file to the S3 bucket in the cloud.

Modules serve as containers that allow to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). One root module can call other child modules and insert their configurations when applying Terraform config. This concept makes the code structure neater, and it allows different team members to work on different parts of configuration at the same time.

First we create a directory named narbyd-project which will be the root-modules inside the root-module we create a directory named modules.

$ mkdir keys-project && cd keys-project

$ mkdir modules

Inside the modules directory, we create the directories that will hold the diiferent resources eg ALB, VPC, ASG, SECGRP, RDS, EFS and also the compute directory.

Copy the files containing the resources that was created in PROJECT-17 into each of the folders created as related to the resources.

In the root-module create a file main.tf, provider.tf, terraform.tfvars, vars.tf file.

The refactored code can be found here.

The main.tf in the root module uses the output.tf files to create the resources for the infrastructure.

Then run the commands:

$ terraform init

$ terraform plan

If we are ok with the actions to be taken we run

$ terraform apply

# Introducing Backend on S3

So far in this project, we have been using the default backend which is the local backend – it requires no configuration and the states file is stored locally. This mode is not a robust solution, so it is better to store it in some more reliable and durable storage.

The problem with storing this file locally is that, in a team of multiple DevOps engineers, other engineers will not have access to a state file stored locally on your computer.

To solve this, we will need to configure a backend where the state file can be accessed remotely by other DevOps team members. There are plenty of different standard backends supported by Terraform that you can choose from. Since we are already using AWS – we can choose an S3 bucket as a backend.

Another useful option that is supported by S3 backend is State Locking – it is used to lock your state file for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state. State Locking feature for S3 backend is optional and requires another AWS service – DynamoDB.

# Setting up the S3 bucket

The steps to Re-initialize Terraform to use S3 backend:

- Add S3 and DynamoDB resource blocks before deleting the local state file
- Update terraform block to introduce backend and locking
- Re-initialize terraform
- Delete the local tfstate file and check the one in S3 bucket
- Add outputs
- terraform apply

Add the below code to the _main.tf file in the root module

resource "aws_s3_bucket" "terraform_state" {
  bucket = "narbyd-dev-terraform-bucket"
  # Enable versioning so we can see the full revision history of our state files
  versioning {
    enabled = true
  }
  # Enable server-side encryption by default
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}

Terraform stores Passwords and secret keys processed by resources in the state files. Hence, we should enable encryption with server_side_encryption_configuration in the above code.

Next, we will create a DynamoDB table to handle locks and perform consistency checks. In previous projects, locks were handled with a local file as shown in terraform.tfstate.lock.info. Since we now have a team mindset, causing us to configure S3 as our backend to store state file, we will do the same to handle locking. Therefore we will use a cloud storage database like DynamoDB so that anyone running Terraform against the same infrastructure can use a central location to control a situation where Terraform is running at the same time from multiple different people.

Dynamo DB resource for locking and consistency checking:

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "narbyd-terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  attribute {
    name = "LockID"
    type = "S"
  }
}

Terraform expects that both S3 bucket and DynamoDB resources are already created before we configure the backend. So, let us run terraform apply to provision resources.

The S3 bucket we created earlier in the project

![Snipe 32](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/745ad8fe-12ee-4adf-923c-463b91d5f673)

# Create the dynamoDB table

![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4ccf50d6-ecf4-4ffc-82db-00674c7227cf)

Configure S3 Backend by adding the code snippet to backend.tf

terraform {
  backend "s3" {
    bucket         = "narbyd-dev-terraform-bucket"
    key            = "global/s3/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}


Run

$ terraform init

Confirm you are OK to change the backend by typing yes

DynamoDB table which we created has an entry which includes state file status.

Navigate to the DynamoDB table inside AWS and leave the page open in your browser.

Run

$ terraform plan

Add Terraform Output

Before you run terraform apply let us add an output so that the S3 bucket Amazon Resource Names ARN and DynamoDB table name can be displayed.

Create a new file and name it output.tf and add below code

output "s3_bucket_arn" {
  value       = aws_s3_bucket.terraform_state.arn
  description = "The ARN of the S3 bucket"
}
output "dynamodb_table_name" {
  value       = aws_dynamodb_table.terraform_locks.name
  description = "The name of the DynamoDB table"
}

Then run terraform apply

Before we run the $ terraform destroy command, we need to comment out the backend configurations and run

$ terraform init -migrate-state

to restore the former state of the terraform.tfstate file.

Then we run terraform destroy

To use our new setup, we uncomment the backend configurations and run

$ terraform init

This configures terraform to use the backend.

N/B:

To upload the terraform.tfstate and the lock file to the S3 bucket and dynomoDB table, we run terraform init after provisioning the other resources.
Our application wont work because in our shell script that was passed to launch, some endpoints like the RDS and EFS access points are needed and they have not been created yet. We will employ the use of Ansible and Packer to fix this in Project-20.
We have successfully refactored the code to use modules, configured the S3 bucket and the DynamoDB to hold the terraform.tfstate file and lock files respectively.

This project continues in project 2o











































































































































