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

  ![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5eb71339-6fad-4e7e-9f5e-983ec8e14ff7)


# Step 3: Creating an Internet Gateway

An Internet Gateway in AWS is like a road that connects your city (VPC) to the outside world (the internet). Without this road, people (data) can't come in or go out of your city (VPC). Deep Dive into Internet Gateways To give your public subnet access to the main road (internet), you need an Internet Gateway. This acts like the entrance and exit to your property. We'll show you how to create and attach an Internet Gateway to your VPC.

- After creating a VPC and Subnets the next is Internet Gateway

  Go to VPC > Internet gateways and click "Create internet gateway" put a name tag and click create gateway then Attach the Internet Gateway to the test-vpc

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d4c76ff4-7417-40ee-90d2-2af4b60317a9)

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a4366c3d-42a6-48e0-8f49-b8be4becc5cc)

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/83006c07-db0c-4849-bbf5-ad07f2a1b0b8)
  
# Step 4: Creating And Configuring Route Tables

Now that we have our entrance and exit (Internet Gateway), we need to give directions to our resources. This is done through a Routing Table. It's like a map, guiding your resources on how to get in and out of your VPC.

Let's go to the route table menu and create a route table for the public subnets.

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c6c0e977-5faa-49eb-9d94-b9e411fe7f3c)

Once created, edit the route table, add the default route to the Internet Gateway(IGW)

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/84ed5310-25fb-4336-bf77-223295e32e93)

Next go the Subnet Associations and clik Edit Subnet Associations

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1223d010-8ef9-40af-a7cf-651fa7149531)

That's it! Now that the VPC is ready, you can run an EC2 instance in public subnets if they need Internet access or in private subnets if they don't.

























