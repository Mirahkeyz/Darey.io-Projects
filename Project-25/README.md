# ORCHESTRATING CONTAINERS ACROSS MULTIPLE VIRTUAL SERVERS WITH KUBERNETES. PART 1

# Container Orchestration with Kubernetes

Container orchestration is a process for automating the deployment, scaling, management, and networking of containers. Here are two key considerations to keep in mind regarding Docker containers:

Ephemeral Nature: Unlike virtual machines, Docker containers are not intended for long-term operation. They are designed to be ephemeral, meaning they can be easily stopped and destroyed. However, you can recreate a new container from the same Docker image with minimal setup and configuration requirements.

Scalability Across Compute Nodes: To achieve high scalability, Docker containers must be configured to run across multiple compute nodes, which can be either physical servers or virtual machines with the Docker engine installed to manage containers.

Now, let's address a specific scenario based on these considerations:

Consider a situation where you have two compute nodes to run your containers. For instance, Node 1 is hosting a container for a Website, while Node 2 is running a database container. In this setup, two important challenges arise:

Resilience Across Nodes: If a container on Node 1 were to fail or become unavailable, it is essential to ensure that it can be automatically restarted on Node 2. Achieving this kind of resilience typically requires additional orchestration tools like Kubernetes, Docker Swarm or other container orchestration solutions. These tools can monitor the health of containers and automatically redistribute workloads across available nodes to maintain service availability.

Inter-Node Communication: Unlike when containers run on the same host, containers on separate hosts do not have native network communication. To enable communication between the Tooling website container on Node 1 and thedatabase container on Node 2, you need to set up networking solutions specifically designed for cross-node communication. This typically involves creating a custom network overlay that spans both hosts. Orchestration tools mentioned earlier often provide networking features to facilitate this communication.

In summary, while Docker containers offer flexibility and ease of deployment, managing containers across multiple compute nodes introduces challenges related to resilience and inter-node communication. To address these challenges, container orchestration tools and network overlays are commonly employed to ensure the reliable and coordinated operation of containers in distributed environments.

Container orchestration is a concept that allows to address these two scenarios, it provides automation of all the aspects of coordinating and managing containers. Container orchestration is focused on managing life cycle of containers and their dynamic environments.

At its core, container orchestration entails the automation of the complete container lifecycle, orchestrating their deployment across multiple nodes, encompassing tasks such as:

- Configuring and efficiently scheduling containers on nodes.
- Ensuring container availability, even in the face of failures.
- Dynamically scaling containers to evenly distribute application workloads across the infrastructure.
- Resource allocation management among containers.
- Implementing load balancing, traffic routing, and enabling service discovery for containers.
- Continuous health monitoring of containers.
- Ensuring robust security measures for container interactions.
- Kubernetes emerges as a formidable tool designed explicitly for container orchestration, excelling when configured correctly.

# Kubernetes From-Ground-Up

For a better understanding of each aspect of spinning up a Kubernetes cluster, I will install each and every component manually from scratch.

To successfully implement "K8s From-Ground-Up", the following and even more will be done as a K8s administrator:

- Install and configure master (also known as control plane) node components and worker nodes.
- Apply security settings across the entire cluster (i.e., encrypting the data in transit, and at rest)
- In transit encryption means encrypting communications over the network using HTTPS
- At rest encryption means encrypting the data stored on a disk
- Plan the capacity for the backend data store etcd
- Configure network plugins for the containers to communicate
- Manage periodical upgrade of the cluster
- Configure observability and auditing

Tools to be used and expected result.

- VM: AWS EC2
- OS: Ubuntu 20.04 lts+
- Docker Engine
- kubectl console utility
- cfssl and cfssljson utilities
- Kubernetes cluster

I will create 3 EC2 Instances, and in the end, I will have the following parts of the cluster properly configured:

- One Master node/Control plane.
- Two Worker Nodes
- Configured SSL/TLS certificates for Kubernetes components to communicate securely
- Configured Node Network
- Configured Pod Network

# SETTING UP THE KUBERNETES CLUSTER (MANUALLY)

INSTALL CLIENT TOOLS

Spin up an EC2 Instance and install some tools. This instance will be the client workstation:

awscli – is a unified tool to manage your AWS services.

kubectl – this command line utility will be the main control tool to manage the K8s cluster.

cfssl – an open source toolkit for everything TLS/SSL from Cloudflare

cfssljson – a program, which takes the JSON output from the cfssl and writes certificates, keys, CSRs, and bundles to disk.

# Install and configure AWS CLI

I need to create a user in my AWS console with programmatic access keys configured in AWS Identity and Access Management (IAM) then Configure AWS CLI to access all AWS services used.

Generate access keys and store them in a safe place.

On my local workstation download and install AWS CLI.

$ sudo apt update && sudo apt install awscli -y

To configure the AWS CLI run:

$ aws configure and follow the prompt.

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a6829348-afb1-447c-8272-0fc717019750)

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/19378ce2-b9b6-48cc-a210-5a42c56387f7)

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/342dfae0-2a9d-4ff5-bca7-472928bd2710)

To verify run aws cli commands.

$ aws ec2 describe-vpcs

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cde8db4e-48b2-4984-acd9-ef6094e34af4)

# Install kubectl

A Kubernetes cluster features a Web API capable of handling HTTP/HTTPS requests. However, continually using curl to send commands can be cumbersome. To streamline the tasks of a Kubernetes administrator, the kubectl command tool was created.

This versatile tool simplifies interactions with Kubernetes, enabling administrators to effortlessly deploy applications, inspect and manage cluster resources, access logs and execute various administrative tasks.

Download the binary

$ wget https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl

Make it executable

$ chmod +x kubectl

Move to the Bin directory

$ sudo mv kubectl /usr/local/bin/

Verify that kubectl version 1.21.0 or higher is installed:

$ kubectl version --client

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e1e60314-e4a9-41e9-8a18-22da0296202f)

# Install CFSSL and CFSSLJSON

cfssl is an open source tool by Cloudflare used to setup a Public Key Infrastructure. for generating, signing and bundling TLS certificates. In previous projects you have experienced the use of Letsencrypt for the similar use case. Here, cfssl will be configured as a Certificate Authority which will issue the certificates required to spin up a Kubernetes cluster.

Download, install and verify successful installation of cfssl and cfssljson:

$ wget -q --show-progress --https-only --timestamping https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssl_1.6.3_linux_amd64 

https://github.com/cloudflare/cfssl/releases/download/v1.6.3/cfssljson_1.6.3_linux_amd64

$ chmod +x cfssl_1.6.3_linux_amd64 cfssljson_1.6.3_linux_amd64

$ sudo mv cfssl_1.6.3_linux_amd64 /usr/local/bin/cfssl

$ sudo mv cfssljson_1.6.3_linux_amd64 /usr/local/bin/cfssljson

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0dd08ea9-a797-4282-8370-00b387ab98c7)

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1f1e6acf-2400-4cd2-a5f5-1a324a9d227e)

# AWS CLOUD RESOURCES FOR KUBERNETES CLUSTER

Configure Network Infrastructure - Virtual Private Cloud (VPC)

Create a directory named manual-k8s-cluster

Create a VPC and store the ID as a variable:

$ VPC_ID=$(aws ec2 create-vpc --cidr-block 172.31.0.0/16 --output text --query 'Vpc.VpcId')

Tag the VPC so that it is named

$ aws ec2 create-tags --resources ${VPC_ID} --tags Key=Name,Value=manual-k8s-cluster

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7c9dfd44-f219-4226-914c-bdc21539c3a9)

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8c2ee1b8-9d66-474e-92d6-95e8e714f58b)

Domain Name System – DNS

Enable DNS support for your VPC

$ aws ec2 modify-vpc-attribute --vpc-id ${VPC_ID} --enable-dns-support '{"Value": true}'

Enable DNS support for hostnames

$ aws ec2 modify-vpc-attribute --vpc-id ${VPC_ID} --enable-dns-hostnames '{"Value": true}'

AWS Region

Set the required region

$ AWS_REGION=us-east-1

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e1615b40-4aff-405e-a6a7-dc578f354c50)

Dynamic Host Configuration Protocol – DHCP

Configure DHCP Options Set

The Dynamic Host Configuration Protocol (DHCP) is a network management protocol employed in Internet Protocol networks. Its primary function is to autonomously allocate IP addresses and establish various communication parameters for devices linked to the network through a client-server architecture.

AWS upon the initiation of VPC, automatically generates and associates a DHCP option set. This option set contains two predefined settings: domain-name-servers, which defaults to AmazonProvidedDNS, and domain-name, which defaults to the domain name associated with your specified region. AmazonProvidedDNS denotes an Amazon Domain Name System (DNS) server, facilitating DNS-based communication for instances.

By default, Amazon Elastic Compute Cloud (EC2) instances are assigned fully qualified domain names, such as ip-172-50-197-106.eu-central-1.compute.internal. However, you can configure your own custom settings, as demonstrated in the example below.

$ DHCP_OPTION_SET_ID=$(aws ec2 create-dhcp-options --dhcp-configuration "Key=domain-name,Values=$AWS_REGION.compute.internal" "Key=domain-name-servers,Values=AmazonProvidedDNS" --output text --query 'DhcpOptions.DhcpOptionsId')

Tag the DHCP Option set

$ aws ec2 create-tags --resources ${DHCP_OPTION_SET_ID} --tags Key=Name,Value=manual-k8s-cluster

Associate the DHCP Option set with the VPC

$ aws ec2 associate-dhcp-options --dhcp-options-id ${DHCP_OPTION_SET_ID} --vpc-id ${VPC_ID}

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2c0058d0-da09-495a-a545-ad9e55c96327)

































































