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

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/afad54af-1ae3-4082-af59-cbdac6bc7fe7)

N/B: We should always endeavour to make our work dynamic. Notice that we have declared multiple resource blocks for each subnet in the code. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a count argument.

Now let us refactor the code.

First we destroy the resources that have been created

$ terraform destroy

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/83f57a9a-0f99-43f3-9795-083aa9ef60e6)

# REFACTORING THE CODE

We will introduce variables, and remove hard coding. In the provider block, declare a variable named region, give it a default value, and update the provider section by referring to the declared variable.

Create vars.tf and terraform.tfvars files

The vars.tf file will contain the variables while the main.tf file will contain the VPC and the provider codes.

Open the vars.tf file and add the following code snippet

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
    default ="true" 
}

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/49327d01-1a77-44f8-b1e7-8a477ef71334)

Terraform has a functionality that allows us to pull data which exposes information to us. For example, every region has Availability Zones (AZ). Different regions have from 2 to 4 Availability Zones. With over 20 geographic regions and over 70 AZs served by AWS, it is impossible to keep up with the latest information by hard coding the names of AZs. Hence, we will explore the use of Terraform’s Data Sources to fetch information outside of Terraform. In this case, from AWS.

Let us fetch Availability zones from AWS, and replace the hard coded value in the subnet’s availability_zone section.

Update the main.tf file with the code


 Get list of availability zones
data "aws_availability_zones" "available" {
    state = "available"
}

To make use of this new data resource, we will need to introduce a count argument in the subnet block.

 Create public subnet1
resource "aws_subnet" "public" {
    count                   = 2
    vpc_id                  = aws_vpc.narbyd-vpc.id
    cidr_block              = "172.16.1.0/24"
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

}

The count tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets. The data resource will return a list object that contains a list of AZs. Internally, Terraform will receive the data like this:

["us-east-1a", "us-east-1b"]

Each of them is an index, the first one is index 0, while the other is index 1. If the data returned had more than 2 records, then the index numbers would continue to increment. i.e 0, 1, 2, 3 ....

Therefore, each time Terraform goes into a loop to create a subnet, it must be created in the retrieved AZ from the list. Each loop will need the index number to determine what AZ the subnet will be created. That is why we refer to data.aws_availability_zones.available.names[count.index] as the value for availability_zone. When the first loop runs, the first index will be 0, therefore the AZ will be us-east-1a. The pattern will repeat for the second loop andif there are more records then the loop continues.

If we run Terraform with this configuration, it may succeed for the first time, but by the time it goes into the second loop, it will fail because we still have cidr_block hard coded. The same cidr_block cannot be created twice within the same VPC. So, we we need to make cidr_block dynamic.

Make cidr_block dynamic

We will introduce a function cidrsubnet() to make this happen. It accepts 3 parameters.

 Create public subnet1
resource "aws_subnet" "public" { 
    count                   = 2
    vpc_id                  = aws_vpc.narbyd-vpc.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]

}

A cidrsubnet function works like an algorithm to dynamically create a subnet CIDR per AZ. Regardless of the number of subnets created, it takes care of the cidr value per subnet.

Its parameters are cidrsubnet(prefix, newbits, netnum)

The prefix parameter i.e var.vpc_cidr must be given in CIDR notation, same as for VPC. The newbits parameter is the number of additional bits with which to extend the prefix. For example, if given a prefix ending with /16 and a newbits value of 4, the resulting subnet address will have length /20. The netnum parameter is a whole number that can be represented as a binary integer with no more than newbits binary digits, which will be used to populate the additional bits added to the prefix.

We can see how this works by entering the terraform console and keep changing the figures to see the output.

On the terminal, run

$ terraform console

type in

> cidrsubnet("172.16.0.0/16", 4, 0)

click Enter.

See the output

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/12ae0611-1bcd-476c-991c-b46a7e0c988f)

Keep changing the numbers and see what happens. Then use exit to exit the console.

The next step is to remove the hard coded count value.

Remove hard coded count value.

Since the data resource returns all the AZs within a region, it makes sense to count the number of AZs returned and pass that number to the count argument.

To do this, we can introuduce length() function, which basically determines the length of a given list, map, or string.

Since data.aws_availability_zones.available.names returns a list like ["us-east-1a", "us-east-1b", "us-east-1c"], we can pass it into a lenght function and get number of the AZs.

Now we can simply update the public subnet block like this

 Create public subnet1
resource "aws_subnet" "public" { 
    count                   = length(data.aws_availability_zones.available.names)
    vpc_id                  = aws_vpc.keys-VPC.id
    cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
    map_public_ip_on_launch = true
    availability_zone       = data.aws_availability_zones.available.names[count.index]
}

This will create the subnet resource required. But the required number of subnets is 2 subnets.

To fix this, we declare a variable to store the desired number of public subnets, and set the default value

variable "preferred_number_of_public_subnets" {
  default = 2
}

We then update the count argument with a condition. Terraform needs to check first if there is a desired number of subnets. Otherwise, use the data returned by the lenght function. See how that is presented below.

 Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.keys-VPC.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

- The var.preferred_number_of_public_subnets == null checks if the value of the variable is set to null or has some value defined.

- The second part ? and length(data.aws_availability_zones.available.names) means if preferred number of public subnets is null (Or not known) then set the value to the data returned by lenght function.

- The : and var.preferred_number_of_public_subnets means if the first condition is false, i.e preferred number of public subnets is not null then set the value to whatever is definied in var.preferred_number_of_public_subnets.

Open the terraform.tfvars file and set values for each of the variables.

region = "us-east-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

preferred_number_of_public_subnets = 2

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3fa43664-dc65-4065-a8c3-75af03c37f5d)

The main.tf should look like this

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e252f078-7b74-4c6f-bf65-7bd7ef22caba)


Run

terraform init

terraform plan

terraform apply

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ed1e0636-22d0-4c78-ac19-6cfef94ecced)

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5f831d29-b7cf-48e4-b062-af58ff45445c)

We have successfully created our VPC and subnets in the region and availability zones respectively.

Run

$ terraform destroy 

to delete the resources.
















































































































