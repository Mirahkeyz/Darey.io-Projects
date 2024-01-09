# INFRASTRUCTURE AUTOMATION WITH IAC USING TERRAFORM (PART 2)

This is a countinuation of Project-17

In this Project, we will continue creating the resources for the AWS setup. The resources to be created include:

- 4 Private subnets
- 1 Internet Gateway
- 1 NAT Gateway
- 1 Elastic IP
- 2 Route tables
- IAM roles
- Security Groups
- Target Group for Nginx, WordPress and Tooling
- Certificate from AWS certificate manager
- External Application Load Balancer and Internal Application Load Balancer.
- Launch template for Bastion, Tooling, Nginx and WordPress
- Auto Scaling Group (ASG) for Bastion, Tooling, Nginx and WordPress
- Elastic Filesystem
- Relational Database (RDS)

# CREATE 4 PRIVATE SUBNETS AND TAGGING

We will create 4 subnets by updating the main.tf with the following code.

resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}

We add the "+ 2" to the code for the count.index in the private subnets so that it doesnt overlap with the public subnets created.

Then update the vars.tf with the following for the private subnets indicating the number of subnts to be created - in our case we need to create 4 subnets.


variable "preferred_number_of_private_subnets" {
  type = 4
  description = "Number of private subnets"
}

Update our main.tf code with the following. Each section of the codes for the private and public subnets should be updated with this

tags = merge(
    var.tags,
    {
      Name = format("PrivateSubnet-%s", count.index)
    } 
  )

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/97f5e2d7-a924-4f5a-9390-717ccc5128dd)

Then update the vars.tf with the following

variable "tags" {
  description = "A mapping of tags to assign to all resources."
  type        = map(string)
  default     = {}
}

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cfff1c6a-7b76-4a90-a5f4-6caa48dffcf0)

And the terraform.tfvars with the tags

tags = {
  Owner-Email     = "miracleanunobi80@gmail.com"
  Managed-By      = "terraform"
  Billing-Account = "153600809351"
} 

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a7f12f95-d5e4-4261-9e36-21e4b21945eb)

So our codes now looks like this -

For the main.tf

provider "aws" {
  region = var.region
}

 Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support
  enable_dns_hostnames           = var.enable_dns_hostnames

  tags = merge(
    var.tags,
    {
      Name = format("%s-VPC", var.name)
    }
  )
}

 Get list of availability zones
data "aws_availability_zones" "available" {
  state = "available"
}

 Create public subnets
resource "aws_subnet" "public" {
  count                   = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
      Name = format("%s-PublicSubnet-%s", var.name, count.index)
    },
  )
}

 Create private subnets
resource "aws_subnet" "private" {
  count                   = var.preferred_number_of_private_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_private_subnets
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index + 2)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
  tags = merge(
    var.tags,
    {
      Name = format("%s-PrivateSubnet-%s", var.name, count.index)
    },
  )

}

The %s takes the interpolated value of var.name while the second %s takes the value of the count.index.

For the vars.tf

variable "region" {
  default = "us-east-1"
}

variable "vpc_cidr" {
  default = "172.16.0.0/16"
}

variable "enable_dns_support" {
  default = "true"
}

variable "enable_dns_hostnames" {
  default = "true"
}

variable "preferred_number_of_public_subnets" {
  type        = number
  description = "Number of public subnets"
}

variable "preferred_number_of_private_subnets" {
  type        = number
  description = "Number of private subnets"
}

variable "tags" {
  description = "A mapping of tags to assign to all resources"
  type        = map(string)
  default     = {}

}

For the terraform.tfvars

region = "us-east-1"

vpc_cidr = "10.0.0.0/16"

enable_dns_support = "true"

enable_dns_hostnames = "true"

preferred_number_of_public_subnets = 2

preferred_number_of_private_subnets = 4

tags = {
  Owner-Email     = "miracleanunobi80@gmail.com"
  Managed-By      = "terraform"
  Billing-Account = "153600809351"
} 


Now we run

$ terraform init

$ terraform plan

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f006a917-6092-458c-8eb2-ada6b6287cbe)




















































