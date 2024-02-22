# DEPLOYING APPLICATIONS INTO KUBERNETES CLUSTER

In this project, I will initiate the deployment of applications within a Kubernetes (K8s) cluster. Kubernetes is a complex system with numerous components, working with multiple layers of abstraction that separate your application from the underlying host machines where it is executed.

we will explore and witness the following aspects in action:

Implementing the deployment of software applications using YAML manifest files, featuring various Kubernetes objects, including:

- Pods
- ReplicaSets
- Deployments
- StatefulSets
- Services (ClusterIP, NodeIP, Loadbalancer)
- Configmaps
- Volumes
- PersistentVolumes
- PersistentVolumeClaims
- And more.
- Understanding the distinctions between stateful and stateless applications.

- Demonstrating the deployment of MySQL as a StatefulSet and providing a rationale for this choice.

- Identifying the limitations associated with deploying applications directly using YAML manifests in Kubernetes.

- Introducing Helm templates, exploring their components, and highlighting the significance of semantic versioning.

- Converting all the existing .yaml templates into a Helm chart for more streamlined management.

- Deploying additional tools on AWS Elastic Kubernetes Service (EKS) using Helm charts, which include:

- Jenkins
- MySQL
- Ingress Controllers (Nginx)
- Cert-Manager
- Ingress configurations for Jenkins and the primary application
- Deploying Monitoring Tools, such as Prometheus and Grafana.
- Exploring the concept of Hybrid CI/CD by integrating various tools like Gitlab CI/CD and Jenkins. Additionally, we'll delve into GitOps principles using Weaveworks Flux.

When utilizing a Kubernetes cluster, the available options vary depending on its intended purpose.

Numerous organizations choose Managed Service solutions for various compelling reasons, such as:

- Less administrative overheads
- Reduced cost of ownership
- Improved Security
- Seamless support
- Periodical updates to a stable and well-tested version
- Faster cluster spin up
  
However, there is usually strong reasons why organisations with very strict compliance and security concerns choose to build their own Kubernetes clusters. Most of the companies that go this route will mostly use on-premises data centres. When there is need to store data privately due to its sensitive nature, companies will rather not use a public cloud provider. Because, if they do, they have no idea of the physical location of the data centre in which their data is being persisted. Banks and Governments are typical examples of this.

Some setup options can combine both public and private cloud together. For example, the master nodes, etcd clusters, and some worker nodes that run stateful applications can be configured in private datacentres, while worker nodes that require heavy computations and stateless applications can run in public clouds. This kind of hybrid architecture is ideal to satisfy compliance, while also benefiting from other public cloud capabilities.

We will be using the Elastic Kubernates Service(EKS) for this project. To set up the EKS, we need to install WSL. To install WSL click here.

OR

Run the following to enable WSL

dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7fde3b5f-f832-4637-8ab5-2ec94e138023)

Reboot your system.

After the reboot, open the Microsoft Store, search for your preferred Linux distribution (e.g., Ubuntu), and install it.

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/31e8d093-eb62-4a4f-8114-566d64cfd0d7)

Complete the initial setup of the Linux distribution by creating a user and password.

Update the ubuntu packages On the WSL

$ sudo apt update && sudo pat upgrade

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4598230f-6e4c-431d-bef1-c8a47e64d016)

Download and install eksctl

```
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```
$ eksctl version

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7f723516-5999-45b7-a675-c09c70794e7b)

For the WSL to interact with the AWS we need to install awscli and configure

$ sudo apt install awscli

$ aws configure

Install pip

$ sudo apt install python3-pip

Upgrade the awscli

$ pip install --upgrade awscli

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c784e230-d5b3-4e2c-ac76-88aef8977036)

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0a1c1570-9f3a-44ef-bc67-eafb170fd242)

To verify run any aws command

$ aws s3 ls

Setup kubectl

To setup kubectl we will refer to the kubernetes documentation.

Install kubectl

$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

Download the kubectl checksum file

$ curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

Validate the kubectl binary against the checksum file

$ echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check

Install kubectl

$ sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

Verify

$ kubectl version --client

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d30b07f0-f26e-4a58-94a7-9c841850b014)

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/18a101e9-f0d8-4ae6-9d06-ed2867a34f65)

Now, you have eksctl installed on your Windows system through WSL. You can use it to interact with Amazon EKS clusters.

```
$ eksctl create cluster \
  --name deploy \
  --region us-east-1 \
  --nodegroup-name worker \
  --node-type t2.micro \
  --nodes 2
```

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/40f3e307-3621-4690-8b55-8588de0e7f88)

# Configure kubectl

After the cluster is created, you need to configure kubectl to connect to the cluster. Run the command

$ aws eks --region us-east-1 update-kubeconfig --name deploy

Then run to get nodes

$ kubectl get nodes

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a7f252af-a4b7-42d6-bd78-e24c65b38452)

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f33b83f6-90ae-4b03-966c-3e45f9ed16d0)

# Creating A Pod For The Nginx Application

Create nginx pod by applying the manifest file nginx-pod.yml manifest file shown below

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels: 
    app: nginx-pod
spec:
  containers:
  - image: nginx:latest
    name: nginx-pod
    ports:
    - containerPort: 80
      protocol: TCP
```

Create the pods

$ kubectl apply -f nginx-pod.yml

We can access the pod using

$ kubectl get pod nginx-pod.yml

To access the information about the pod

$ kubectl describe pod nginx-pod.yml

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c4651e40-f234-4f77-b008-0729ed10ccfc)

# ACCESSING THE APP FROM THE BROWSER

The primary objective of any solution is to enable access through either a web portal or an application, such as a mobile app. In our current setup, we have a Pod equipped with an Nginx container. However, this Pod can't be accessed directly from a web browser due to its unique IP address.

To resolve this issue, we introduce another Kubernetes component known as a "Service."

A Service acts as an intermediary that receives requests and forwards them to the respective Pod's IP address.

In essence, a Service acts as a gateway that accepts incoming requests on behalf of the Pods and routes them to the appropriate Pod's IP address. If you execute the provided command, you can obtain the IP address of the Pod. Nonetheless, it's important to note that there is no direct means of accessing this Pod from the external world.

$ kubectl get pod nginx-pod  -o wide

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/027beccc-974e-474f-a1fd-b24c223ebff5)

# Expose a Service on a server’s public IP address & static port

Sometimes, it may be needed to directly access the application using the public IP of the server (when we speak of a K8s cluster we can replace ‘server’ with ‘node’) the Pod is running on. This is when the NodePort service type comes in handy.

A Node port service type exposes the service on a static port on the node’s IP address. NodePorts are in the 30000-32767 range by default, which means a NodePort is unlikely to match a service’s intended port (for example, 80 may be exposed as 30080).

Create a Service - nginx-svc.yml manifest file

```
 apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: NodePort
  selector:
    app: nginx-pod
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30080
```

Create the service

$ kubectl apply -f nginx-svc.yml

We can access the svc using

$ kubectl get svc -o wide

To access the information about the service

$ kubectl describe svc nginx-service

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/711feb72-8d08-4ec4-a0bf-994c61faf2ba)

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dae4159b-5041-4d8c-ab0f-2a41f442d2a6)

To access the service,

Allow the inbound traffic in your EC2’s Security Group to the NodePort range 30080

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5f3eb617-c580-47f0-ba41-dfd71e65ca0c)

Access the nginx using the public IP address of the node the Pod is running on

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/857124b4-475f-4196-b6e0-42f2eff06ec5)

The port number 30080 designates the specific port associated with the node where the Pod is currently scheduled to operate. Should the Pod undergo rescheduling to a different node, it will retain this very port number on its new hosting node. Consequently, if you have multiple Pods concurrently running on diverse nodes, each of them will be accessible via their respective node IP addresses, all employing the same consistent port number.

To delete the pod

$ kubectl delete nginx-pod

To delete the service

$ kubectl delete nginx-service

CREATE A REPLICA SET

Let us create a rs.yml manifest for a ReplicaSet object

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx-pod
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      name: nginx-pod
      labels:
         app: nginx-pod
         tier: frontend
    spec:
      containers:
      - image: nginx:latest
        name: nginx-pod
        ports:
        - containerPort: 80
          protocol: TCP
```

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/df30250b-e53e-4815-ae4c-b6b564c2fabb)

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dbf784ff-7b99-45c0-89f3-51380d6d2893)

















































































































































































