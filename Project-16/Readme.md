# AWS NETWORKING IMPLEMENTATION

# What is an Amazon VPC?

An Amazon Virtual Private Cloud (VPC) is like your own private section of the Amazon cloud, where you can place and manage your resources (like servers or databases). You control who and what can go in and out, just like a gated community.

# Step 1: Creating A New VPC

Go to your AWS Console and search for VPC then click on create VPC.  we'll use the "VPC and more" option later so choose "VPC only" . Enter any name like "test-vpc" as the name tag and "10.0.0.0/16" as the IPv4 CIDR. The "10.0.0.0/16" will be the primary IPv4 block and you can add a secondary IPv4 block e.g., "100.64.0.0/16". The use case of secondary CIDR block could be because you're running out of IPs and need to add additional block, or there's a VPC with overlapping CIDR which you need to peer or connect. See this blog post on how a secondary CIDR block is being used in an overlapping IP scenario

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2354e4bc-e364-4397-9684-049ef7e4ab2d)

# Step 2: Creating And Configuring Subnets

Subnets are like smaller segments example, you can't proceed as it requires within a VPC that help you organize and manage your resources. Subnets are like dividing an office building into smaller sections, where each section represents a department. In this analogy, subnets are created to organize and manage the network effectively.

Below is how to create a subnet

- After creating a VPC click on Subnet on the VPC dashboard then click on create subnets...we are going to create four subnets. 2 Public subnets and 2 Private subnets. follow the screenshot below

  Public-subnet1a = 10.0.11.0/24 us-east-1a

  Public-subnet2b = 10.0.12.0/24 us-east-1b

  Private-subnet1a = 10.0.1.0/24 us-east-2a

  Private-subnet2b = 10.0.2.0/24 us-east-2b

  ![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/feecccec-1923-4ec8-abe9-96af348cbd7a)



























