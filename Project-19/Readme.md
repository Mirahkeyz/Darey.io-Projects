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














































