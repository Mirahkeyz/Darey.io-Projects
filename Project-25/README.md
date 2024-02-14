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

Subnet

Create and tag the Subnet

$ SUBNET_ID=$(aws ec2 create-subnet --vpc-id ${VPC_ID} --cidr-block 172.31.0.0/24 --output text --query 'Subnet.SubnetId')

$ aws ec2 create-tags --resources ${SUBNET_ID} --tags Key=Name,Value=manual-k8s-cluster

Internet Gateway – IGW

Create the Internet Gateway and attach it to the VPC

$ INTERNET_GATEWAY_ID=$(aws ec2 create-internet-gateway --output text --query 'InternetGateway.InternetGatewayId')

$ aws ec2 create-tags --resources ${INTERNET_GATEWAY_ID} --tags Key=Name,Value=manual-k8s-cluster

Attah to VPC

$ aws ec2 attach-internet-gateway --internet-gateway-id ${INTERNET_GATEWAY_ID} --vpc-id ${VPC_ID}

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4f1f2fab-abfb-4513-a096-25a57a66a83c)

Route tables

Create route tables, associate the route table to subnet, and create a route to allow external traffic to the Internet through the Internet Gateway

$ ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id ${VPC_ID} --output text --query 'RouteTable.RouteTableId')

$ aws ec2 create-tags --resources ${ROUTE_TABLE_ID} --tags Key=Name,Value=manual-k8s-cluster

$ aws ec2 associate-route-table --route-table-id ${ROUTE_TABLE_ID} --subnet-id ${SUBNET_ID}

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ae449e10-1eda-4801-9f04-62f33894e40e)

Create a route to allow external traffic to the Internet through the Internet Gateway

$ aws ec2 create-route --route-table-id ${ROUTE_TABLE_ID} --destination-cidr-block 0.0.0.0/0 --gateway-id ${INTERNET_GATEWAY_ID}

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c2ad7030-5e7f-4772-9dc0-4ce35fc66e24)

Security Groups

Configure security groups

Create the security group and store its ID in a variable

$ SECURITY_GROUP_ID=$(aws ec2 create-security-group --group-name manual-k8s-cluster --description "Kubernetes cluster security group" --vpc-id ${VPC_ID} --output text --query 'GroupId')

Create the NAME tag for the security group

$ aws ec2 create-tags --resources ${SECURITY_GROUP_ID} --tags Key=Name,Value=manual-k8s-cluster

Create Inbound traffic for all communication within the subnet to connect on ports used by the master node(s)

$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --ip-permissions IpProtocol=tcp,FromPort=2379,ToPort=2380,IpRanges='[{CidrIp=172.31.0.0/24}]'

Create Inbound traffic for all communication within the subnet to connect on ports used by the worker nodes

$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --ip-permissions IpProtocol=tcp,FromPort=30000,ToPort=32767,IpRanges='[{CidrIp=172.31.0.0/24}]'

Create inbound traffic to allow connections to the Kubernetes API Server (port 6443) listening on port 6443

$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 6443 --cidr 0.0.0.0/0

Create inbound SSH traffic from any source. In a production environment, restrict access exclusively to desired IPs or CIDR ranges for connection.

$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol tcp --port 22 --cidr 0.0.0.0/0

Create ICMP ingress for all types

$ aws ec2 authorize-security-group-ingress --group-id ${SECURITY_GROUP_ID} --protocol icmp --port -1 --cidr 0.0.0.0/0

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7de5c245-7330-45be-92bb-1d63a8b5a63c)

Network Load Balancer

Create a network Load balancer

$ LOAD_BALANCER_ARN=$(aws elbv2 create-load-balancer --name manual-k8s-cluster --subnets ${SUBNET_ID} --scheme internet-facing --type network --output text --query 'LoadBalancers[].LoadBalancerArn')

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/50ed0397-7857-4b7d-8dba-86a4ef80b1bb)

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a1c83e20-4580-4d2f-a3d9-cbba12206a84)

Tagret Group

Create a target group acknowledging its lack of defined criteria at this stage due to the absence of real targets. it will be "unhealthy".

$ TARGET_GROUP_ARN=$(aws elbv2 create-target-group --name manual-k8s-cluster --protocol TCP --port 6443 --vpc-id ${VPC_ID} --target-type ip --output text --query 'TargetGroups[].TargetGroupArn')

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f63897fc-6e41-449e-8610-7f64bbb5da72)

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dbde78c2-fff9-47c7-91b8-d353a0667be6)

Register targets: Similar to above, you will provide the IP addresses for registration purposes. These IP addresses will serve as targets when the nodes become available.

$ aws elbv2 register-targets --target-group-arn ${TARGET_GROUP_ARN} --targets Id=172.31.0.1{0,1,2}

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/272ffde7-e3a1-49a1-9eca-0d76696b0ffb)

Create a listener to listen for requests and forward to the target nodes on TCP port 6443

$ aws elbv2 create-listener --load-balancer-arn ${LOAD_BALANCER_ARN} --protocol TCP --port 6443 --default-actions Type=forward,TargetGroupArn=${TARGET_GROUP_ARN} --output text --query 'Listeners[].ListenerArn'

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/62cfe665-7bd4-492f-af30-7408374554a0)

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/add004b6-55e2-43c5-8868-e622928516f1)

K8s Public Address

Get the Kubernetes Public address

$ KUBERNETES_PUBLIC_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5c3a1f36-5c78-45fc-ad67-6008c83a6def)

CREATE COMPUTE RESOURCES

Install jq tool

jq is a lightweight and flexible command-line tool for processing and manipulating JSON data. It is commonly used in Unix-like operating systems to filter, transform, and format JSON data from various sources, including files, APIs, and other data streams. jq provides a wide range of features for querying and manipulating JSON, making it a powerful tool for tasks like parsing JSON, extracting specific data, and creating new JSON structures.

$ sudo apt update && sudo apt install jq -y

![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2353e5f1-1016-4e2c-a021-d9c5a3fe3141)

AMI

Get an image to create EC2 instances

$ IMAGE_ID=$(aws ec2 describe-images --owners 099720109477 --filters 'Name=root-device-type,Values=ebs' 'Name=architecture,Values=x86_64' 'Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*' | jq -r '.Images|sort_by(.Name)[-1]|.ImageId')

![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a6f34de7-9e2c-4180-80f7-bb2d083d63d9)

SSH key-pair

Create SSH Key-Pair

$ mkdir -p ssh

$ aws ec2 create-key-pair --key-name manual-k8s-cluster --output text --query 'KeyMaterial' > ssh/manual-k8s-cluster.id_rsa

$ chmod 600 ssh/manual-k8s-cluster.id_rsa

![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bf6addaa-0345-43b7-8d8a-7049d40e79bc)

EC2 Instances for Controle Plane (Master Nodes)

Create 3 Master nodes

```
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name manual-k8s-cluster \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.1${i} \
    --user-data "name=master-${i}" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=manual-k8s-cluster-master-${i}"
done
```

![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7c064822-0d60-4c0e-8fe0-e7819bfc40c1)

![Snipe 28](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d326d3ac-3a16-489a-9122-a1466e95b911)

![Snipe 29](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5bd39e54-64bc-4b61-9b45-34558fda4d01)

EC2 Instances for Worker Nodes

Create 3 worker nodes

```
for i in 0 1 2; do
  instance_id=$(aws ec2 run-instances \
    --associate-public-ip-address \
    --image-id ${IMAGE_ID} \
    --count 1 \
    --key-name manual-k8s-cluster \
    --security-group-ids ${SECURITY_GROUP_ID} \
    --instance-type t2.micro \
    --private-ip-address 172.31.0.2${i} \
    --user-data "name=worker-${i}|pod-cidr=172.20.${i}.0/24" \
    --subnet-id ${SUBNET_ID} \
    --output text --query 'Instances[].InstanceId')
  aws ec2 modify-instance-attribute \
    --instance-id ${instance_id} \
    --no-source-dest-check
  aws ec2 create-tags \
    --resources ${instance_id} \
    --tags "Key=Name,Value=manual-k8s-cluster-worker-${i}"
done
```

# PREPARE THE SELF-SIGNED CERTIFICATE AUTHORITY AND GENERATE TLS CERTIFICATES

The following components running on the Master node will require TLS certificates.

kube-controller-manager
kube-scheduler
etcd
kube-apiserver
The following components running on the Worker nodes will require TLS certificates.

kubelet
kube-proxy
Therefore, you will provision a PKI Infrastructure using cfssl which will have a Certificate Authority. The CA will then generate certificates for all the individual component

Self-Signed Root Certificate Authority (CA)

Here, We will provision a CA that will be used to sign additional TLS certificates.

Create a directory and cd into it

$ mkdir ca-authority && cd ca-authority

```
{

cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": ["signing", "key encipherment", "server auth", "client auth"],
        "expiry": "8760h"
      }
    }
  }
}
EOF

cat > ca-csr.json <<EOF
{
  "CN": "Kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca

}
```

![Snipe 31](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/682153c7-cc74-4f5f-85ff-c6285beebb90)

![Snipe 32](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/74c60e68-8f74-4f1e-95c5-02e42af8ab37)

The 3 important files here are:

- ca.pem – The Root Certificate
- ca-key.pem – The Private Key
- ca.csr – The Certificate Signing Request
  
Generating TLS Certificates For Client and Server

We will need to provision Client/Server certificates for all the components. We must have encrypted communication within the cluster. Therefore, the server here are the master nodes running the api-server component. While the client is every other component that needs to communicate with the api-server.

Now we have a certificate for the Root CA, we can then begin to request more certificates which the different Kubernetes components, i.e. clients and server, will use to have encrypted communication.

Remember, the clients here refer to every other component that will communicate with the api-server. These are:

- kube-controller-manager
- kube-scheduler
- etcd
- kubelet
- kube-proxy
- Kubernetes Admin User

Let us begin with the Kubernetes API-Server Certificate and Private Key

The certificate for the Api-server must have IP addresses, DNS names, and a Load Balancer address included. Otherwise, we will have a lot of difficulties connecting to the api-server.

Generate the Certificate Signing Request (CSR), Private Key and the Certificate for the Kubernetes Master Nodes.

```
{
cat > master-kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
   "hosts": [
   "127.0.0.1",
   "172.31.0.10",
   "172.31.0.11",
   "172.31.0.12",
   "ip-172-31-0-10",
   "ip-172-31-0-11",
   "ip-172-31-0-12",
   "ip-172-31-0-10.${AWS_REGION}.compute.internal",
   "ip-172-31-0-11.${AWS_REGION}.compute.internal",
   "ip-172-31-0-12.${AWS_REGION}.compute.internal",
   "${KUBERNETES_PUBLIC_ADDRESS}",
   "kubernetes",
   "kubernetes.default",
   "kubernetes.default.svc",
   "kubernetes.default.svc.cluster",
   "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  master-kubernetes-csr.json | cfssljson -bare master-kubernetes
}
```

![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1b06f11d-daea-419f-b346-7bc3e487465d)

Creating the other certificates: for the following Kubernetes components:

- Scheduler Client Certificate
- Kube Proxy Client Certificate
- Controller Manager Client Certificate
- Kubelet Client Certificates
- K8s admin user Client Certificate

kube-scheduler Client - Certificate and Private Key
```
{

cat > kube-scheduler-csr.json <<EOF
{
  "CN": "system:kube-scheduler",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-scheduler",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-scheduler-csr.json | cfssljson -bare kube-scheduler

}
```

kube-proxy Client - Certificate and Private Key

```
{

cat > kube-proxy-csr.json <<EOF
{
  "CN": "system:kube-proxy",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:node-proxier",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-proxy-csr.json | cfssljson -bare kube-proxy

}
```

kube-controller-manager - Client Certificate and Private Key

```
{
cat > kube-controller-manager-csr.json <<EOF
{
  "CN": "system:kube-controller-manager",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:kube-controller-manager",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager

}
```

kubelet Client - Certificate and Private Key

Similar to how we configured the api-server's certificate, Kubernetes requires that the hostname of each worker node is included in the client certificate.

Also, Kubernetes uses a special-purpose authorization mode called Node Authorizer, that specifically authorizes API requests made by kubelet services. In order to be authorized by the Node Authorizer, kubelets must use a credential that identifies them as being in the system:nodes group, with a username of system:node:. Notice the "CN": "system:node:${instance_hostname}", in the below code.

Therefore, the certificate to be created must comply to these requirements. In the below example, there are 3 worker nodes, hence we will use bash to loop through a list of the worker nodes’ hostnames, and based on each index, the respective Certificate Signing Request (CSR), private key and client certificates will be generated.

```
for i in 0 1 2; do
  instance="manual-k8s-cluster-worker-${i}"
  instance_hostname="ip-172-31-0-2${i}"
  cat > ${instance}-csr.json <<EOF
{
  "CN": "system:node:${instance_hostname}",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:nodes",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')

  internal_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PrivateIpAddress')

  cfssl gencert \
    -ca=ca.pem \
    -ca-key=ca-key.pem \
    -config=ca-config.json \
    -hostname=${instance_hostname},${external_ip},${internal_ip} \
    -profile=kubernetes \
    manual-k8s-cluster-worker-${i}-csr.json | cfssljson -bare manual-k8s-cluster-worker-${i}
done
```

kubernetes admin user - Client Certificate and Private Key

```
{
cat > admin-csr.json <<EOF
{
  "CN": "admin",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "system:masters",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  admin-csr.json | cfssljson -bare admin
}
```

Token Controller - certificate and private key

We need to generate certificate and private key for the Token Controller - a part of the Kubernetes Controller Manager.

kube-controller-manager responsible for generating and signing service account tokens which are used by pods or other resources to establish connectivity to the api-server.

```
{

cat > service-account-csr.json <<EOF
{
  "CN": "service-accounts",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "UK",
      "L": "England",
      "O": "Kubernetes",
      "OU": "Dybran-projects",
      "ST": "London"
    }
  ]
}
EOF

cfssl gencert \
  -ca=ca.pem \
  -ca-key=ca-key.pem \
  -config=ca-config.json \
  -profile=kubernetes \
  service-account-csr.json | cfssljson -bare service-account
}
```

![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/750626f4-5df3-4619-b19d-72e2389042e8)

DISTRIBUTING THE CLIENT AND SERVER CERTIFICATES

Now it is time to start sending all the client and server certificates to their respective instances.

Let us begin with the worker nodes:

Copy these files securely to the worker nodes using scp utility

- Root CA certificate – ca.pem
- X509 Certificate for each worker node
- Private Key of the certificate for each worker node

```
for i in 0 1 2; do
  instance="manual-k8s-cluster-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    ca.pem ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```
![Snipe 35](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/864e85a1-f9ea-4a8f-9550-d12921721566)

Master or Controller node: – Note that only the api-server related files will be sent over to the master nodes.

```
for i in 0 1 2; do
instance="manual-k8s-cluster-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    master-kubernetes.pem master-kubernetes-key.pem ubuntu@${external_ip}:~/;
done
```

![Snipe 36](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fd891972-019f-4487-93c4-32ab4deb6590)

The kube-proxy, kube-controller-manager, kube-scheduler and kubelet client certificates will be used to generate client authentication configuration files later on.

GENERATE KUBERNETES CONFIGURATION FILES FOR AUTHENTICATION USING 'KUBECTL'

In this step, We will create some files known as kubeconfig, which enables Kubernetes clients to locate and authenticate to the Kubernetes API Servers.

You will need a client tool called kubectl to do this. And, by the way, most of your time with Kubernetes will be spent using kubectl commands.

Now it’s time to generate kubeconfig files for the kubelet, controller manager, kube-proxy, and scheduler clients and then the admin user.

First, let us create a few environment variables for reuse by multiple commands.

$ KUBERNETES_API_SERVER_ADDRESS=$(aws elbv2 describe-load-balancers --load-balancer-arns ${LOAD_BALANCER_ARN} --output text --query 'LoadBalancers[].DNSName')

Generate the kubelet kubeconfig file

For each of the nodes running the kubelet component, it is very important that the client certificate configured for that node is used to generate the kubeconfig. This is because each certificate has the node’s DNS name or IP Address configured at the time the certificate was generated. It will also ensure that the appropriate authorization is applied to that node through the Node Authorizer

Run the code below in the directory where all the certificates were generated.

```
for i in 0 1 2; do

instance="manual-k8s-cluster-worker-${i}"
instance_hostname="ip-172-31-0-2${i}"

 # Set the kubernetes cluster in the kubeconfig file
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://$KUBERNETES_API_SERVER_ADDRESS:6443 \
    --kubeconfig=${instance}.kubeconfig

# Set the cluster credentials in the kubeconfig file
  kubectl config set-credentials system:node:${instance_hostname} \
    --client-certificate=${instance}.pem \
    --client-key=${instance}-key.pem \
    --embed-certs=true \
    --kubeconfig=${instance}.kubeconfig

# Set the context in the kubeconfig file
  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:node:${instance_hostname} \
    --kubeconfig=${instance}.kubeconfig

  kubectl config use-context default --kubeconfig=${instance}.kubeconfig
done
```

![Snipe 37](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2489c402-0835-4336-b808-1ff0995d487c)

List the output

$ ls -ltr *.kubeconfig

![Snipe 38](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7abc1cd9-4bad-40b8-b924-70275fad190c)

Open up the kubeconfig files generated and review the 3 different sections that have been configured:

- Cluster
- Credentials
- Kube Context

Kubeconfig file is used to organize information about clusters, users, namespaces and authentication mechanisms. By default, kubectl looks for a file named config in the $HOME/.kube directory. You can specify other kubeconfig files by setting the KUBECONFIG environment variable or by setting the --kubeconfig flag.

Context part of kubeconfig file defines three main parameters: cluster, namespace and user. You can save several different contexts with any convenient names and switch between them when needed.

$  kubectl config use-context %context-name%

Generate the kube-proxy kubeconfig

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-credentials system:kube-proxy \
    --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

  kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
}
```

![Snipe 39](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3c6c2b9f-d7f2-40f3-9837-78221909c1b0)

Generate the Kube-Controller-Manager kubeconfig

Notice that the --server is set to use 127.0.0.1. This is because, this component runs on the API-Server so there is no point routing through the Load Balancer.

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:kube-controller-manager \
    --kubeconfig=kube-controller-manager.kubeconfig

  kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
}
```

![Snipe 40](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6fee1aa8-6f0e-43eb-b4cb-7c03d9bd46b8)

Generating the Kube-Scheduler Kubeconfig

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://127.0.0.1:6443 \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem \
    --client-key=kube-scheduler-key.pem \
    --embed-certs=true \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

  kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
}
```
![Snipe 41](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/45e942b6-a7d7-4010-bcf7-ad5290990f01)

Generate the kubeconfig file for the admin user

```
{
  kubectl config set-cluster manual-k8s-cluster \
    --certificate-authority=ca.pem \
    --embed-certs=true \
    --server=https://${KUBERNETES_API_SERVER_ADDRESS}:6443 \
    --kubeconfig=admin.kubeconfig

  kubectl config set-credentials admin \
    --client-certificate=admin.pem \
    --client-key=admin-key.pem \
    --embed-certs=true \
    --kubeconfig=admin.kubeconfig

  kubectl config set-context default \
    --cluster=manual-k8s-cluster \
    --user=admin \
    --kubeconfig=admin.kubeconfig

  kubectl config use-context default --kubeconfig=admin.kubeconfig
}
```
![Snipe 42](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/01a357fa-4873-4aba-b729-f379c54b69ab)

Distribute the files to their respective servers using scp and a for loop.

For Worker nodes

```
for i in 0 1 2; do
  instance="manual-k8s-cluster-worker-${i}"
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    kube-proxy.kubeconfig ${instance}-key.pem ${instance}.pem ubuntu@${external_ip}:~/; \
done
```

![Snipe 43](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/869dbc66-378d-43f7-8ade-3d3ab55d0f46)


For Master nodes

```
for i in 0 1 2; do
instance="manual-k8s-cluster-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ../ssh/manual-k8s-cluster.id_rsa \
    ca.pem ca-key.pem service-account-key.pem service-account.pem \
    kube-controller-manager.kubeconfig kube-scheduler.kubeconfig admin.kubeconfig ubuntu@${external_ip}:~/;
done
```

![Snipe 44](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a261c350-260e-4f82-a96a-0c14b978d9f6)

# PREPARE THE ETCD DATABASE FOR ENCRYPTION AT REST

Prepare the etcd database for encryption at rest.

Kubernetes uses etcd (A distributed key value store) to store variety of data which includes the cluster state, application configurations, and secrets. By default, the data that is being persisted to the disk is not encrypted. Any attacker that is able to gain access to this database can exploit the cluster since the data is stored in plain text. Hence, it is a security risk for Kubernetes that needs to be addressed.

To mitigate this risk, we must prepare to encrypt etcd at rest. "At rest" means data that is stored and persists on a disk. Anytime you hear "in-flight" or "in transit" refers to data that is being transferred over the network. "In-flight" encryption is done through TLS.

Generate the encryption key and encode it using "base64"

Kubernetes uses a 32-byte (256-bit) encryption key for etcd encryption. Etcd is a distributed key-value store used by Kubernetes to store configuration data and other distributed system information securely. The 32-byte key is typically used for encryption in etcd to ensure the confidentiality and integrity of the stored data. This key is generated and managed as part of the Kubernetes cluster's configuration for security purposes.

$ ETCD_ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)

See the output

$ echo $ETCD_ENCRYPTION_KEY

![Snipe 45](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8493201e-e75d-4a8a-958a-8aa1de854727)

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ETCD_ENCRYPTION_KEY}
      - identity: {}
EOF
```

![Snipe 46](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1b419fc2-a794-4e6d-9706-af43212a90fe)

Send the encryption file to the Controller nodes using scp and a for loop

```
for i in 0 1 2; do
instance="manual-k8s-cluster-master-${i}" \
  external_ip=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=${instance}" \
    --output text --query 'Reservations[].Instances[].PublicIpAddress')
  scp -i ssh/manual-k8s-cluster.id_rsa \
    encryption-config.yaml ubuntu@${external_ip}:~/;
done
```
![Snipe 47](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7d0e4a94-bf22-41f0-96da-82e7aa0854ef)

# Bootstrap "etcd" cluster

The primary purpose of the etcd component is to store the state of the cluster. This is because Kubernetes itself is stateless. Therefore, all its stateful data will persist in etcd. Since Kubernetes is a distributed system – it needs a distributed storage to keep persistent data in it. etcd is a highly-available key value store that fits the purpose. All K8s cluster configurations are stored in a form of key value pairs in etcd, it also stores the actual and desired states of the cluster. etcd cluster is intelligent enough to watch for changes made on one instance and almost instantly replicate those changes to the rest of the instances, so all of them will be always reconciled.

SSH into the controller server

Open three terminals in MobaXterm, SSH into the client-workstation and from each of these terminals, establish SSH connections to the different master nodes. 

Master 1

```
master_1_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-master-0" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${master_1_ip}
```

Master 2

```
master_2_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-master-1" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${master_2_ip}
```

Master 3

```
master_3_ip=$(aws ec2 describe-instances \
--filters "Name=tag:Name,Values=manual-k8s-cluster-master-2" \
--output text --query 'Reservations[].Instances[].PublicIpAddress')
ssh -i ssh/manual-k8s-cluster.id_rsa ubuntu@${master_3_ip}
```
Run the command

$ ls -ltr

You should be able to see all the files that have been sent to the nodes

Download and install etcd on each of the terminals using the multi-paste button.

$ wget -q --show-progress --https-only --timestamping \ "https://github.com/etcd-io/etcd/releases/download/v3.4.15/etcd-v3.4.15-linux-amd64.tar.gz"

Extract and install the etcd server and the etcdctl command line utility

```
{
tar -xvf etcd-v3.4.15-linux-amd64.tar.gz
sudo mv etcd-v3.4.15-linux-amd64/etcd* /usr/local/bin/
}
```

Configure the etcd server

```
{
  sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem master-kubernetes-key.pem master-kubernetes.pem /etc/etcd/
}
```

The instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance

$ export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)

Each etcd member must have a unique name within an etcd cluster. Set the etcd name to node Private IP address so it will uniquely identify the machine

```
$ ETCD_NAME=$(curl -s http://169.254.169.254/latest/user-data/ \
  | tr "|" "\n" | grep "^name" | cut -d"=" -f2)

echo ${ETCD_NAME}
```

Create the etcd.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster master-0=https://172.31.0.10:2380,master-1=https://172.31.0.11:2380,master-2=https://172.31.0.12:2380 \\
  --cert-file=/etc/etcd/master-kubernetes.pem \\
  --key-file=/etc/etcd/master-kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/master-kubernetes.pem \\
  --peer-key-file=/etc/etcd/master-kubernetes-key.pem \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Start and enable the etcd Server

```

{
sudo systemctl daemon-reload
sudo systemctl enable etcd
sudo systemctl start etcd
}
```

Verify the etcd installation

```
sudo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/master-kubernetes.pem \
  --key=/etc/etcd/master-kubernetes-key.pem
```

Check the status

$ sudo systemctl status etcd

![Snipe 48](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a7d54f67-f460-42ac-90b7-ff199972bbd9)

# BOOTSTRAP THE CONTROL PLANE

Configure the components for the control plane on the master/controller nodes.

Create the Kubernetes configuration directory

$ sudo mkdir -p /etc/kubernetes/config

Download the official Kubernetes release binaries:

```
wget -q --show-progress --https-only --timestamping \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-apiserver" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-controller-manager" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kube-scheduler" \
"https://storage.googleapis.com/kubernetes-release/release/v1.21.0/bin/linux/amd64/kubectl"
```

Install the Kubernetes binaries

```
{
chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl
sudo mv kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/local/bin/
}
```

Configure the Kubernetes API Server

```
{
sudo mkdir -p /var/lib/kubernetes/

sudo mv ca.pem ca-key.pem master-kubernetes-key.pem master-kubernetes.pem \
service-account-key.pem service-account.pem \
encryption-config.yaml /var/lib/kubernetes/
}
```

The instance internal IP address will be used to advertise the API Server to members of the cluster. Retrieve the internal IP address for the current compute instance:

$ export INTERNAL_IP=$(curl -s http://169.254.169.254/latest/meta-data/local-ipv4)

Create the kube-apiserver.service systemd unit file Ensure to read each startup flag used in below systemd file from the documentation here

```
cat <<EOF | sudo tee /etc/systemd/system/kube-apiserver.service
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \\
  --advertise-address=${INTERNAL_IP} \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/audit.log \\
  --authorization-mode=Node,RBAC \\
  --bind-address=0.0.0.0 \\
  --client-ca-file=/var/lib/kubernetes/ca.pem \\
  --enable-admission-plugins=NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --etcd-cafile=/var/lib/kubernetes/ca.pem \\
  --etcd-certfile=/var/lib/kubernetes/master-kubernetes.pem \\
  --etcd-keyfile=/var/lib/kubernetes/master-kubernetes-key.pem\\
  --etcd-servers=https://172.31.0.10:2379,https://172.31.0.11:2379,https://172.31.0.12:2379 \\
  --event-ttl=1h \\
  --encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \\
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \\
  --kubelet-client-certificate=/var/lib/kubernetes/master-kubernetes.pem \\
  --kubelet-client-key=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --runtime-config='api/all=true' \\
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \\
  --service-account-signing-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-account-issuer=https://${INTERNAL_IP}:6443 \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --service-node-port-range=30000-32767 \\
  --tls-cert-file=/var/lib/kubernetes/master-kubernetes.pem \\
  --tls-private-key-file=/var/lib/kubernetes/master-kubernetes-key.pem \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
Configure the Kubernetes Controller Manager

Move the kube-controller-manager kubeconfig into place

$ sudo mv kube-controller-manager.kubeconfig /var/lib/kubernetes/

Export some variables to retrieve the vpc_cidr – This will be required for the bind-address flag

```
export AWS_METADATA="http://169.254.169.254/latest/meta-data"
export EC2_MAC_ADDRESS=$(curl -s $AWS_METADATA/network/interfaces/macs/ | head -n1 | tr -d '/')
export VPC_CIDR=$(curl -s $AWS_METADATA/network/interfaces/macs/$EC2_MAC_ADDRESS/vpc-ipv4-cidr-block/)
export NAME=manual-k8s-cluster
```
Create the kube-controller-manager.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/kube-controller-manager.service
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \\
  --bind-address=0.0.0.0 \\
  --cluster-cidr=${VPC_CIDR} \\
  --cluster-name=${NAME} \\
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \\
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \\
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authentication-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --authorization-kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \\
  --leader-elect=true \\
  --root-ca-file=/var/lib/kubernetes/ca.pem \\
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \\
  --service-cluster-ip-range=172.32.0.0/24 \\
  --use-service-account-credentials=true \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Configure the Kubernetes Scheduler

Move the kube-scheduler kubeconfig into place

$ sudo mv kube-scheduler.kubeconfig /var/lib/kubernetes/

$ sudo mkdir -p /etc/kubernetes/config

Create the kube-scheduler.yaml configuration file

```
cat <<EOF | sudo tee /etc/kubernetes/config/kube-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1beta1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
EOF
```

Create the kube-scheduler.service systemd unit file

```
cat <<EOF | sudo tee /etc/systemd/system/kube-scheduler.service
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \\
  --config=/etc/kubernetes/config/kube-scheduler.yaml \\
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

Start the Controller Services

```
{
sudo systemctl daemon-reload
sudo systemctl enable kube-apiserver kube-controller-manager kube-scheduler
sudo systemctl start kube-apiserver kube-controller-manager kube-scheduler
}
```

Check the status of the services. Start with the kube-scheduler and kube-controller-manager. It may take up to 20 seconds for kube-apiserver to be fully loaded.

```
{
sudo systemctl status kube-apiserver
sudo systemctl status kube-controller-manager
sudo systemctl status kube-scheduler
}
```


![Snipe 49](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0d7d81c0-44e3-47be-955a-9e46343963ed)

![Snipe 50](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/31e19860-afb2-4df8-b339-75302966b8da)






















































































































































































































































































































































































































































































































































































































































































































































































































