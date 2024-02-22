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


The manifest file of ReplicaSet consist of the following fields

apiVersion: This field specifies the version of kubernetes Api to which the object belongs. ReplicaSet belongs to apps/v1 apiVersion.
kind: This field specify the type of object for which the manifest belongs to. Here, it is ReplicaSet.
metadata: This field includes the metadata for the object. It mainly includes two fields: name and labels of the ReplicaSet.
spec: This field specifies the label selector to be used to select the Pods, number of replicas of the Pod to be run and the container or list of containers which the Pod will run. In the above example, we are running 3 replicas of nginx container.
Create the nginx replicaset

$ kubectl apply -f nginx-rs.yml

We can access the pods using

$ kubectl get pods

To access the information about the pods

$ kubectl describe pod <pod-id>

OR

$ kubectl get pod <pod-id> -o yaml

Detailed information about the replicaset

$ kubectl get rs nginx-rs -o yaml

OR

$ kubectl get rs nginx-rs -o json

We can easily scale our ReplicaSet by specifying the desired number of replicas

$ kubectl scale rs nginx-rs --replicas=<number-of-pods>

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dd8b7879-a4aa-41cc-8456-001bd5363791)

# Advanced label matching

As Kubernetes mature as a technology, so does its features and improvements to k8s objects. ReplicationControllers do not meet certain complex business requirements when it comes to using selectors. Imagine if you need to select Pods with multiple lables that represents things like

- Application tier: such as Frontend, or Backend
- Environment: such as Dev, SIT, QA, Preprod or Prod

So far, we used a simple selector that just matches a key-value pair and check only equality

```
  selector:
    app: nginx-pod
```
    
But in some cases, we want ReplicaSet to manage our existing containers that match certain criteria, we can use the same simple label matching or we can use some more complex conditions, such as:

- in
- not in
- not equal
- etc... Let us create the rs.yml manifest file

```
apiVersion: apps/v1
kind: ReplicaSet
metadata: 
  name: nginx-rs
spec:
  replicas: 2
  selector:
    matchLabels:
      env: prod
    matchExpressions:
    - { key: tier, operator: In, values: [frontend] }
  template:
    metadata:
      name: nginx
      labels: 
        env: prod
        tier: frontend
    spec:
      containers:
      - name: nginx-container
        image: nginx:latest
        ports:
        - containerPort: 80
          protocol: TCP
```

In the above spec file, under the selector, matchLabels and matchExpression are used to specify the key-value pair. The matchLabel works exactly the same way as the equality-based selector, and the matchExpression is used to specify the set based selectors. This feature is the main differentiator between ReplicaSet and previously mentioned obsolete ReplicationController.

Create the replicaset

$ kubectl apply -f rs.yml

Get the replication set

$ kubectl get rs nginx-rs -o wide

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/917646c6-31a3-46c2-86a8-b4e7890007ac)

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/38845fc3-dc6c-4948-b515-9eaab0526977)

# USING AWS LOAD BALANCER TO ACCESS YOUR SERVICE IN KUBERNETES.

We have previously interacted with the Nginx service using ClusterIP and NodeIP. However, there's yet another service type known as LoadBalancer. This particular service not only establishes a Service object within Kubernetes but also sets up an actual external Load Balancer, such as the Elastic Load Balancer (ELB) in AWS, if available.

Create the service and ensure that the selector references the Pods in the replica set.

```
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: LoadBalancer
  selector:
    tier: frontend
  ports:
    - protocol: TCP
      port: 80 # This is the port the Loadbalancer is listening at
      targetPort: 80 # This is the port the container is listening at
```

Create the nginxlb-svc.yml

$ kubectl apply -f nginxlb-svc.yml

An ELB resource will be created in your AWS console

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f487631a-8c80-4b65-aeab-7424dfb565a2)

![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0f69a1a6-ac81-4183-8d5b-0696ced0ba18)

![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6f589eec-69c3-4e3e-b119-4f8db443d7eb)

To get the endpoint for the load balancer

$ kubectl get svc

Access the nginx from the browser

![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/16bf866a-a0e7-406a-b804-d89a36884878)

To get information about the nginx-service

$ kubectl describe svc nginx-service

OR

$ kubectl get svc nginx-service -o yaml

![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a99ea737-87b9-481a-8eae-443f2f6c18e6)

A clusterIP key is updated in the manifest and assigned an IP address. Even though you have specified a Loadbalancer service type, internally it still requires a clusterIP to route the external traffic through. In the ports section, nodePort is still used. This is because Kubernetes still needs to use a dedicated port on the worker node to route the traffic through. Ensure that port range 30000-32767 is opened in your inbound Security Group configuration.

Delete the replicaset and the nginx service

$ kubectl delete rs nginx-rs

$ kubectl delete svc nginx-service

USING DEPLOYMENT CONTROLLERS

A Deployment is another layer above ReplicaSets and Pods, newer and more advanced level concept than ReplicaSets. It manages the deployment of ReplicaSets and allows for easy updating of a ReplicaSet as well as the ability to roll back to a previous version of deployment. It is declarative and can be used for rolling updates of micro-services, ensuring there is no downtime.

Officially, it is highly recommended to use Deployments to manage replica sets rather than using replica sets directly.

Create a deploy.yml manifest

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  labels:
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```
![Snipe 28](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fdc2d222-2077-4be7-b4d6-1c8311a087a9)

Create the deployment

$ kubectl apply -f deploy.yaml

Get the deployment

$ kubectl get deploy

Get the replicaset

$ kubectl get rs

Get the pods

$ kubectl get pods

![Snipe 29](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ea3043a3-c342-4ec9-af85-50f747ec480c)

From the above we will find that one f the pods is pending. To get information about the pod

$ kubectl describe pod <pod-id>

![Snipe 30](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7f19937d-beb3-415f-8a52-63c9dd874e5a)

The Event will help us with the problem with the pod.

The warning indicates that there are no available nodes to schedule your pod because all nodes are occupied because I used t2.micro to create the clsuter.

Some of the ways to resolve this include:

Add more nodes to your cluster.
Adjust resource requests and limits.
Prioritize pods.
Use PodDisruptionBudgets.
Scale down or remove unnecessary pods.
I had to update the depoy.yml file to scale down the replicaset to 2.

Connect into one of the Pods' container to run Linux commands

$ kubectl exec -it nginx-deploy-7d476d754d-fcd55 /bin/bash

List the files and folders in the Nginx directory

# ls -latr /etc/nginx

We can access the content of the default.conf

# cat /etc/nginx/conf.d/default.conf

![Snipe 31](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/72118115-021b-4d1c-82b5-78def8a7e2c2)

Create the nginxlb-svc.yml service and access the nignx from the browser using the loadbalancer endpoint.

![Snipe 32](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8c59ad4e-24fc-4cf4-bc38-c38863482089)

# PERSISTING DATA FOR PODS

Deployments are stateless by design. Hence, any data stored inside the Pod’s container does not persist when the Pod dies.

If you were to update the content of the index.html file inside the container, and the Pod dies, that content will not be lost since a new Pod will replace the dead one.

To see this in action, scale down the pods from 2 to 1

$ kubectl scale deployment nginx-deploy --replicas=1

![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e07fd8b5-2301-44c7-97d6-414019675d01)

Connect into the pod and install vim

$ kubectl exec -itnginx-deploy-7d476d754d-fcd55 /bin/bash

 apt update && apt install vim -y

![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5bb8da20-1086-4110-9f3c-7586eb16db0f)

Update the content of the file and add the code below /usr/share/nginx/html/index.html

$ vim /usr/share/nginx/html/index.html

```
<!DOCTYPE html>
<html>
<head>
<title>Welcome to Miracle's Page!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to Miracle's Page!</h1>
<p>Learning by doing is so much fun</p>

<p>Project Based learning at 
<a href="https://darey.io/">www.darey.io</a>.<br/>
Start your Learning today at
<a href="https://darey.io/">www.darey.io</a>.</p>

<p><em>Thanks</em></p>
</body>
</html>
```

Reload the browser to see the changes made

Now, delete the only running Pod

$ kubectl delete pod nginx-deploy-7d476d754d-fcd55

We will see that the replicaset creates another pod with a different pod ID.



Refresh the web page

![Snipe 35](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/90a63783-75bd-4d0d-9333-39d45b5bdf6d)

![Snipe 36](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/330580a8-a9b4-482a-8c07-d196cf8f3d30)

You will see that the content that was saved in the container is no longer there. That is because Pods do not store data when they are being recreated – that is why they are called ephemeral or stateless.

Storage is a critical part of running containers, and Kubernetes offers some powerful primitives for managing it. Dynamic volume provisioning, a feature unique to Kubernetes, which allows storage volumes to be created on-demand. Without dynamic provisioning, DevOps engineers must manually make calls to the cloud or storage provider to create new storage volumes, and then create PersistentVolume objects to represent them in Kubernetes. The dynamic provisioning feature eliminates the need for DevOps to pre-provision storage. Instead, it automatically provisions storage when it is requested by users.

To make the data persist in case of a Pod’s failure, you will need to configure the Pod to use following objects:

Persistent Volume or pv – is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using Storage Classes.
Persistent Volume Claim or pvc. Persistent Volume Claim is simply a request for storage, hence the "claim" in its name.















































































































































































































































































































































































































