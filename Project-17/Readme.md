![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/54689e4b-932d-4462-a18b-c87247ccd502)![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/de9ed82c-61cd-469f-983a-fa38aaf60b53)![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/58e49680-1cfe-4fbe-b1af-a357a5109803)# AUTOMATING AWS INFRASTRUCTURE IN CODE (TERRAFORM)

# Prerequisites:

- AWS account;
- AWS Identify and Access Management (IAM) credentials and programmatic access.
- Set up AWS credentials locally with aws configure in the AWS Command Line Interface (CLI)

  To write quality Terraform codes, we need to:

- Understand the goal of the desired infrastructure.
- Ability to effectively use up to date Terraform documentation. Click here

The resources to be created include:

S3 buket
A VPC
2 Public subnets

# CREATE S3 BUCKET

Create an S3 bucket to store Terraform state file

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/697b3c4e-8ff2-45ad-ab4e-76866cc2f506)

From my terminal, I should be able to see the S3 bucket i just created if my aws CLI is configured. Run the command

$ aws s3 ls

# CREATING VPC | SUBNETS | SECURITY GROUPS

- Create a directory structure.

In VS Code, Create a folder called PBL and create a file in the folder, name it main.tf

- Install the Terraform on vscode

- Add AWS as a provider, and a resource to create a VPC in the main.tf file. The provider block informs Terraform that we intend to build infrastructure within AWS while the resource block will create a VPC.

provider "aws" {
  region = "us-east-1"
}

 Create VPC
resource "aws_vpc" "keys-VPC" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
  tags = {
    Name = "narbyd-VPC"
  }
}

Run the command on your vscode terminal

terraform init

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ae95acbc-6042-4036-b707-ef6b1d8a011b)

A new file has been created: .terraform. This is where Terraform keeps plugins.

we should check to see what terraform intends to create before we tell it to go ahead and create it by running the command

terraform plan

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/89c993e6-9e39-458d-990c-7b0d380783e1)

If we are good with the chages planned, run the command

terraform apply

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/09cb7492-bff9-4f86-8b34-4ec6a4650269)

![Snipe 4 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f430d2f0-7505-4dd6-99e2-ad7767ca4f04)

This creates the VPC - keys-VPC and its attributes.

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f1954a53-dd96-4224-a703-e25c51b37730)

A new file is created terraform.tfstate This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.

If you also observed closely, you would realise that another file - terraform.tfstate.lock.info gets created during planning and apply but this file gets deleted immediately. This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts.

# Subnets resource section

According to our architectural design, we require 6 subnets:

- 2 public
- 2 private for webservers
- 2 private for data layer

Let us create the first 2 public subnets.

Add below configuration to the main.tf file:

 Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.keys-VPC.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"
    tags = {
        Name = "keys-pub-1"
    }

}

 Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.keys-VPC.id
    cidr_block                 = "172.16.2.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"
     tags = {
        Name = "keys-pub-2"
    }
}


![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5d5f6aaf-bbe6-4cf7-9c50-682b350adf47)

We are creating 2 subnets, therefore declaring 2 resource blocks – one for each of the subnets. We are using the vpc_id argument to interpolate the value of the VPC id by setting it to aws_vpc.main.id. This way, Terraform knows inside which VPC to create the subnet.

Run the commands

terraform plan

terraform apply

















