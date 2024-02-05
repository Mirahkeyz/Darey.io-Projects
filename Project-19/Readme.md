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

# WHEN TO USE WORKSPACES OR DIRECTORY?

To separate environments with significant configuration differences, use a directory structure. Use workspaces for environments that do not greatly deviate from each other, to avoid duplication of your configurations. Try both methods in the sections below to help you understand which will serve your infrastructure best.

For now, you can read more about both alternatives here and try both methods yourself, but we will explore them better in next projects.

Security Groups refactoring with dynamic block For repetitive blocks of code you can use dynamic blocks in Terraform.

# Refactor Security Groups creation with dynamic blocks.

EC2 refactoring with Map and Lookup Remember, every piece of work you do, always try to make it dynamic to accommodate future changes. Amazon Machine Image (AMI) is a regional service which means it is only available in the region it was created. But what if we change the region later, and want to dynamically pick up AMI IDs based on the available AMIs in that region? This is where we will introduce Map and Lookup functions.

Map uses a key and value pairs as a data structure that can be set as a default type for variables.

To select an appropriate AMI per region, we will use a lookup function which has following syntax: lookup(map, key, [default]).

Note: A default value is better to be used to avoid failure whenever the map data has no key.

resource "aws_instace" "web" {
    ami  = "${lookup(var.images, var.region), "ami-12323"}
}

Now, the lookup function will load the variable images using the first parameter. But it also needs to know which of the key-value pairs to use. That is where the second parameter comes in. The key us-east-1 could be specified, but then we will not be doing anything dynamic there, but if we specify the variable for region, it simply resolves to one of the keys. That is why we have used var.region in the second parameter.

Conditional Expressions If you want to make some decision and choose some resource based on a condition – you shall use Terraform Conditional Expressions.

In general, the syntax is as following: condition ? true_val : false_val

Read following snippet of code and try to understand what it means:

resource "aws_db_instance" "read_replica" {
  count               = var.create_read_replica == true ? 1 : 0
  replicate_source_db = aws_db_instance.this.id
}


resource "aws_db_instance" "read_replica" {
  count               = var.create_read_replica == true ? 1 : 0
  replicate_source_db = aws_db_instance.this.id
}

- true #condition equals to ‘if true’
- ? #means, set to ‘1`
- : #means, otherwise, set to ‘0’

# Terraform Modules and best practices to structure your .tf codes

By this time, you might have realized how difficult is to navigate through all the Terraform blocks if they are all written in a single long .tf file. As a DevOps engineer, you must produce reusable and comprehensive IaC code structure, and one of the tool that Terraform provides out of the box is Modules.

Modules serve as containers that allow to logically group Terraform codes for similar resources in the same domain (e.g., Compute, Networking, AMI, etc.). One root module can call other child modules and insert their configurations when applying Terraform config. This concept makes your code structure neater, and it allows different team members to work on different parts of configuration at the same time.

You can also create and publish your modules to Terraform Registry for others to use and use someone’s modules in your projects.

Module is just a collection of .tf and/or .tf.json files in a directory.

This is include in my terraform github repository.

![ACM1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7e6a0dd2-042d-4a7c-b4d3-05b5c3fb5572)

![ACM2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9aae4d3a-2fc1-4c51-9d81-f777718c6558)

![ACM3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/88c9e35a-38d3-43c6-a151-4594c3214110)

![ACM4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/29a31bd4-fa51-4ee7-aa58-db0ee683d8f1)

Final Terraform Plan

![ACM5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8317eb90-9d02-484e-976b-32cadad6e599)























































































































































































































