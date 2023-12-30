# AUTOMATING AWS INFRASTRUCTURE IN CODE (TERRAFORM)

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
resource "aws_vpc" "narbyd-vpc" {
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













