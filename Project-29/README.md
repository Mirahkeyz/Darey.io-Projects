# Deploying and Packaging Applications into Kubernetes with Helm

In Project-24, we acquired practical skills in using Helm to deploy applications on Kubernetes.

Now, in this project, we are focusing on deploying a suite of DevOps tools. Our goal is to confront and understand the typical challenges faced in real-world deployments while mastering effective troubleshooting strategies. We will explore the customization of Helm values files to automate application setups. Throughout the process of deploying different DevOps tools, we'll actively interact with them, comprehending their place in the DevOps lifecycle and how they integrate into the larger ecosystem.

Our primary focus will be on:

- Artifactory
- Ingress Controllers
- Cert-Manager

  Then

Prometheus
Grafana
Elasticsearch ELK using ECK.

Artifactory is part of a suit of products from a company called Jfrog. Jfrog started out as an artifact repository where software binaries in different formats are stored. Today, Jfrog has transitioned from an artifact repository to a DevOps Platform that includes CI and CD capabilities. This has been achieved by offering more products in which Jfrog Artifactory is part of. Other offerings include

- JFrog Pipelines - a CI-CD product that works well with its Artifactory repository. Think of this product as an alternative to Jenkins.
- JFrog Xray - a security product that can be built-into various steps within a JFrog pipeline. Its job is to scan for security vulnerabilities in the stored artifacts. It is able to scan all dependent code.


In this project, the requirement is to use Jfrog Artifactory as a private registry for the organisation's Docker images and Helm charts. This requirement will satisfy part of the company's corporate security policies to never download artifacts directly from the public into production systems. We will eventually have a CI pipeline that initially pulls public docker images and helm charts from the internet, store in artifactory and scan the artifacts for security vulnerabilities before deploying into the corporate infrastructure. Any found vulnerabilities will immediately trigger an action to quarantine such artifacts.

# Deploy Jfrog Artifactory into Kubernetes

First, we provision the kubernetes cluster using eksctl. See Project-22.

Create the cluster

$ eksctl create cluster --name dybran-eks-tooling --region us-west-1 --nodegroup-name worker --node-type t3.medium --nodes 2

Create kubeconfig file using awscli and connect to the kubectl.

$ aws eks update-kubeconfig --name dybran-eks-tooling --region us-west-1

Create a namespace tools where all the DevOps tools will be deployed. We will also be deploying jenkins from the previous project in this namespace.

$ kubectl create ns tools

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9b1d09c0-f2af-470a-9c6a-fe3c0b81a19b)

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d43d809a-26cd-4e3a-afb1-5cddb9475b5b)

Create EBS-CSI Driver for the Cluster

An EBS CSI driver is a crucial component in a Kubernetes cluster that utilizes Amazon Elastic Block Store (EBS) for persistent storage. It enables seamless integration between Kubernetes and EBS, allowing for dynamic provisioning, management, and lifecycle control of EBS volumes for containerized applications.

Here are the key reasons why an EBS CSI driver is essential for a Kubernetes cluster:

1. Dynamic Provisioning: The EBS CSI driver eliminates the need for manual EBS volume creation and configuration, enabling dynamic provisioning of EBS volumes directly within Kubernetes. This streamlines the storage provisioning process and reduces administrative overhead.

2. Automated Attachment: The EBS CSI driver automatically attaches and detaches EBS volumes to the appropriate Kubernetes nodes based on pod scheduling. This ensures that containers have access to the required storage without manual intervention.

3. Volume Lifecycle Management: The EBS CSI driver manages the entire lifecycle of EBS volumes, including creation, deletion, resizing, and snapshotting. This provides a unified approach to storage management within Kubernetes.

4. Simplified Storage Management: The EBS CSI driver simplifies storage management in Kubernetes by decoupling the storage interface from the Kubernetes controller manager. This allows for more efficient storage management and reduces the complexity of the Kubernetes control plane.

5. Enhanced Storage Flexibility: The EBS CSI driver supports a variety of EBS volume configurations, including different volume types, sizes, and performance options. This provides greater flexibility in tailoring storage to specific application requirements.

6. Integration with Kubernetes Ecosystem: The EBS CSI driver is fully integrated with the Kubernetes ecosystem, including Kubernetes PersistentVolumes, PersistentVolumeClaims, and StorageClasses. This allows for seamless integration with existing Kubernetes storage workflows.

Overall, the EBS CSI driver plays a critical role in enabling Kubernetes clusters to effectively leverage EBS for persistent storage. It simplifies storage management, automates volume lifecycle operations, and enhances storage flexibility, making it an indispensable tool for Kubernetes environments.

# Installing EBS CSI Driver

Run the command to see the pods in the kube-system namespace

$ kubectl get pods -n kube-system

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/99ee181d-28cc-4bb8-9dc5-5040e42239d6)

By running this command, you will observe the presence of coredns, kube-proxy, and aws-nodes.

Once the Ebs-csi driver is installed, additional nodes related to ebs-csi will appear in the kube-system namespace.

Check the link to setup the EBS CSI add-on

Use this command to check the necessary platform version.

$ aws eks describe-addon-versions --addon-name aws-ebs-csi-driver

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b7555f96-b33f-4b8a-9ea9-1e9d2c22e242)

You might already have an AWS IAM OpenID Connect (OIDC) provider for your cluster. To confirm its existence or establish a new one, check the OIDC issuer URL linked to your cluster. An IAM OIDC provider is necessary for utilizing IAM roles with service accounts. You can set up an IAM OIDC provider for your cluster using either eksctl or the AWS Management Console.

To create an IAM OIDC identity provider for your cluster with eksctl

Determine the OIDC issuer ID for your cluster.

Retrieve your cluster's OIDC issuer ID and store it in a variable.

$ cluster_name=miracle-eks-tooling

$ oidc_id=$(aws eks describe-cluster --name $cluster_name --region us-west-1 --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)

$ echo $oidc_id

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/80db7abb-a70f-4729-bd2e-c60d3bbc45f9)

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/740866fe-2a1d-4864-8c47-f4823ad68560)

Check if there's an IAM OIDC provider in your account that matches your cluster's issuer ID.

$ aws iam list-open-id-connect-providers | grep $oidc_id | cut -d "/" -f4

If output is returned, then you already have an IAM OIDC provider for your cluster and you can skip the next step. If no output is returned, then you must create an IAM OIDC provider for your cluster.

In this case, no output was returned.

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c0c058ac-fc7f-442f-906f-33a638d67017)

Create an IAM OIDC identity provider for your cluster with the following command

$ eksctl utils associate-iam-oidc-provider --cluster $cluster_name --region us-west-1 --approve

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/97eeadda-c5fc-4aef-8237-059980fd828f)

Configuring a Kubernetes service account to assume an IAM role

Create a file aws-ebs-csi-driver-trust-policy.json that includes the permissions for the AWS services

```
cat >aws-ebs-csi-driver-trust-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::939895954199:oidc-provider/oidc.eks.us-west-1.amazonaws.com/id/$oidc_id"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-west-1.amazonaws.com/id/$oidc_id:aud": "sts.amazonaws.com",
          "oidc.eks.us-west-1.amazonaws.com/id/$oidc_id:sub": "system:serviceaccount:kube-system:ebs-csi-controller-sa"
        }
      }
    }
  ]
}
EOF
```

This trust policy essentially allows the EBS CSI driver to assume the role associated with the specified OIDC provider and access AWS resources on behalf of the EBS CSI controller service account

Create the role - AmazonEKS_EBS_CSI_DriverRole

```
aws iam create-role \
  --role-name AmazonEKS_EBS_CSI_DriverRole \
  --assume-role-policy-document file://"aws-ebs-csi-driver-trust-policy.json
```

Attach a policy. AWS maintains an AWS managed policy or you can create your own custom policy. Attach the AWS managed policy to the role.

```
aws iam attach-role-policy \
  --policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
  --role-name AmazonEKS_EBS_CSI_DriverRole
```

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/68e921e4-a5c3-4db5-8d9d-b0a90fab2df3)

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6c037517-84c4-486a-b052-791b70266bb4)

To add the Amazon EBS CSI add-on using the AWS CLI

Run the following command.

```
aws eks create-addon --cluster-name $cluster_name --addon-name aws-ebs-csi-driver \
  --service-account-role-arn arn:aws:iam::939895954199:role/AmazonEKS_EBS_CSI_DriverRole --region us-west-1
```

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b59f28a0-de69-4191-adac-f75539d27e4a)

Execute the command, you will find the ebs-csi driver related pods available.

$ kubectl get pods -n kube-system

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d3700813-451f-47ec-aa62-99ad9d3c759e)

Installing the tools in kubernetes

The best approach to easily get tools into kubernetes is to use helm.

Install jenkins in the namespace tools using helm

Search for an official helm chart for jenkins on Artifact Hub.

$ helm repo add jenkins https://charts.jenkins.io

$ helm repo update

$ helm upgrade --install jenkins jenkins/jenkins -n tools

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fa837663-a4ad-498a-8628-aeadaa127028)

Install artifactory in the namespace tools

Search for an official helm chart for Artifactory on Artifact Hub.

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9ed5703c-a6e5-48fa-832f-2169d0facea4)

Click on install to display the commands for installation.

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9d21d3ec-9118-454d-918b-6915382d400f)

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a82635ea-2164-4603-b678-a09a127b6e27)

Add the repo

$ helm repo add jfrog https://charts.jfrog.io

Update the helm repo index on my local machine/laptop

$ helm repo update

Install artifactory in the namespace tools

$ helm upgrade --install artifactory jfrog/artifactory --version 107.71.4 -n tools

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a8b669cc-26c4-459c-9b8e-4f8d98cd6674)

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8ee5758a-49f9-4592-bee2-61e565a4a0ca)

We opted for the upgrade --install flag over helm install artifactory jfrog/artifactory for enhanced best practices, especially in CI pipeline development for helm deployments. This approach guarantees that helm performs an upgrade if an installation exists. In the absence of an existing installation, it conducts the initial install. This strategy assures a fail-safe command; it intelligently discerns whether an upgrade or a fresh installation is needed, preventing failures.

Getting the Artifactory URL

The artifactory helm chart comes bundled with the Artifactory software, a PostgreSQL database and an Nginx proxy which it uses to configure routes to the different capabilities of Artifactory. Getting the pods after some time, you should see something like the below.

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/48ee14fb-2ae8-414f-97b3-b4de482f3569)

Each of the deployed application have their respective services. This is how you will be able to reach either of them.

Notice that, the Nginx Proxy has been configured to use the service type of LoadBalancer. Therefore, to reach Artifactory, we will need to go through the Nginx proxy's service. Which happens to be a load balancer created in the cloud provider.

$ kubectl get svc -n tools

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9e2b2d59-d41c-426b-9646-63d86c384ae5)

Accessing the artifactory using the Load balancer URL

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ca63a9a0-9bbd-4a41-813e-7aab0f62aba9)

Login using default

username: admin

password: password

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6d0017ca-39d7-4d97-8c83-a997afd7c3ef)

How the Nginx URL for Artifactory is configured in Kubernetes

How did Helm configure the URL in kubernetes?

Helm uses the values.yaml file to set every single configuration that the chart has the capability to configure. The best place to get started with an off the shelve chart from artifacthub.io is to get familiar with the DEFAULT VALUES section on Artifact hub.

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8d08f728-622a-4b74-858c-3c01dbe82db3)

Explore key and value pairs within the system.

For instance, entering "nginx" into the search bar will display all the configured options for the nginx proxy. Choosing "nginx.enabled" from the list will promptly navigate you to the corresponding configuration in the YAML file.

![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1a5c3ee2-fec9-43d3-b67c-ada77b04ab61)

Search for nginx.service and choose nginx.service.type. This will display the configured Kubernetes service type for Nginx. By default, it appears as LoadBalancer.

![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/52bc78ed-ae53-40ed-9306-320fb411d631)

![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5dd2010f-61e5-469c-9d90-76a2f82b803b)

To work directly with the values.yaml file, you can download the file locally by clicking on the download icon.

![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c80d45e4-a5f0-41f9-ad61-84b150a2af21)

# Ingress controller

Configuring applications in Kubernetes to be externally accessible often begins with setting the service type to a Load Balancer. However, this can lead to escalating expenses and complex management as the number of applications grows, resulting in a multitude of provisioned load balancers.

An optimal solution lies in leveraging Kubernetes Ingress instead, as detailed in Kubernetes Ingress documentation. Yet, this transition requires deploying an Ingress Controller.

One major advantage of employing an Ingress controller is its ability to utilize a single load balancer across various deployed applications. This consolidation allows for the reuse of the load balancer by services like Artifactory and other tools. Consequently, it significantly reduces cloud expenditure and minimizes the overhead associated with managing multiple load balancers, a topic we'll delve deeper into shortly.

For now, we will leave artifactory, move on to the next phase of configuration (Ingress, DNS(Route53) and Cert Manager), and then return to Artifactory to complete the setup so that it can serve as a private docker registry and repository for private helm charts.

Deploying Ingress Controller and managing Ingress Resources

An ingress in Kubernetes is an API object responsible for overseeing external access to services within the cluster. It handles tasks like load balancing, SSL termination, and name-based virtual hosting. Essentially, an Ingress facilitates the exposure of HTTP and HTTPS routes from outside the cluster to services within cluster. Traffic direction is managed by rules set within the Ingress resource.

An ingress resource for Artifactory would look like this

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: artifactory
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.artifactory.dybran.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory
            port:
              number: 8082
```

- An Ingress needs apiVersion, kind, metadata and spec fields
- The name of an Ingress object must be a valid DNS subdomain name
- Ingress frequently uses annotations to configure some options depending on the Ingress controller.
- Different Ingress controllers support different annotations. Therefore it is important to be up to date with the ingress controller's specific documentation to know what annotations are supported.
- It is recommended to always specify the ingress class name with the spec ingressClassName: nginx. This is how the Ingress controller is selected, especially when there are multiple configured ingress controllers in the cluster.
- The domain dybran.com should be replaced with your own domain which has already been purchased from domain providers and configured in AWS Route53.
- artifactory in the backend field is the name of the artifactory service we already have running.

If you attempt to apply the specified YAML configuration for the ingress resource without an ingress controller, it won't function. For the Ingress resource to operate, the cluster must have an active ingress controller. Unlike various controllers running as part of the kube-controller-manager—like the Node Controller, Replica Controller, Deployment Controller, Job Controller, or Cloud Controller—Ingress controllers don't initiate automatically with the cluster. Kubernetes officially supports and maintains AWS, GCE, and NGINX ingress controllers. However, numerous other 3rd-party Ingress controllers exist, offering similar functionalities alongside their distinct features. Among these, the officially supported ones include:

- AKS Application Gateway Ingress Controller (Microsoft Azure)
- Istio
- Traefik
- Ambassador
- HA Proxy Ingress
- Kong
- Gloo

While there are more 3rd-party Ingress controllers available, the aforementioned ones are currently backed and maintained by Kubernetes. A comparison matrix of these controllers can assist in understanding their unique traits, aiding businesses in choosing the right fit for their requirements. In a cluster, it's feasible to deploy multiple ingress controllers, thanks to the essence of an ingress class. By specifying the spec ingressClassName field on the ingress object, the appropriate ingress controller will be utilized by the ingress resource.

Deploy Nginx Ingress Controller

We will deploy and use the Nginx Ingress Controller. It is always the default choice when starting with Kubernetes projects. It is reliable and easy to use. Since this controller is maintained by Kubernetes, there is an official guide the installation process. Hence, we wont be using artifacthub.io here. Even though you can still find ready to go charts there, it just makes sense to always use the official guide in this scenario.

Using the Helm approach, according to the official guide;

- Install Nginx Ingress Controller in the ingress-nginx namespace

```
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

This command is idempotent. It installs the ingress controller if it is not already present. However, if the ingress controller is already installed, it will perform an upgrade instead.

$ kubectl get pods --namespace=ingress-nginx

The following command will wait for the ingress controller pod to be up, running, and ready

```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=120s
```

OR

We can also install the ingress-nginx using the same approach used in installing jenkins and artifactory.

Create a name space ingress-nginx

$ kubectl create ns ingress-nginx

Search for an official helm chart for ingress-nginx on Artifact Hub.

$ helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

Update repo

$ helm repo update

Install ingress-nginx

$ helm upgrade --install ingress-nginx ingress-nginx/ingress-nginx --version 4.8.3 -n ingress-nginx

![Snipe 28](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/70387634-a186-4036-8812-ffd362c7a733)

Get pods in the ingress-nginx namespace

$ kubectl get pods -n ingress-nginx -w

![Snipe 29](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1ad1cc71-40eb-4327-a47f-73287670f545)

Check to see the created load balancer in AWS.

$ kubectl get svc -n ingress-nginx

![Snipe 30](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b4dfebc6-0cc5-48d6-aed8-18fb025ebd3a)

kubectl get services --all-namespaces

the above command picks the default loadbalancer which is the second as the one to use

The ingress-nginx-controller service that was created is of the type LoadBalancer. That will be the load balancer to be used by all applications which require external access, and is using this ingress controller. If you go ahead to AWS console, copy the address in the EXTERNAL-IP column, and search for the loadbalancer, you will see an output like below.

![Snipe 31](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9dfbc7b4-958b-4150-93ba-0cb3bfa8d275)

Check the IngressClass that identifies this ingress controller.

$ kubectl get ingressclass -n ingress-nginx

# Deploy Artifactory Ingress

Now, it is time to configure the ingress so that we can route traffic to the Artifactory internal service, through the ingress controller's load balancer.

Notice the section with the configuration that selects the ingress controller using the ingressClassName

Create the artifactory-ingress.yaml manifest

``
cat <<EOF > artifactory-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: artifactory
spec:
  ingressClassName: nginx
  rules:
  - host: "tooling.artifactory.dybran.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory-artifactory-nginx
            port:
              number: 80
EOF
```
Create the ingress resource in the tools namespace

$ kubectl apply -f artifactory-ingress.yaml -n tools

Get the ingress resource

$ kubectl get ingress -n tools

![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0a7fec88-23c8-4558-a2e7-cd7cbc125d3d)

Note:

CLASS - The nginx controller class name nginx

HOSTS - The hostname to be used in the browser tooling.artifactory.dybran.com

ADDRESS - The loadbalancer address that was created by the ingress controller

Configure DNS

When accessing the tool, sharing the lengthy load balancer address poses significant inconvenience. The ideal solution involves creating a DNS record that's easily readable by humans and capable of directing requests to the balancer. This exact configuration is set within the ingress object as host: "tooling.artifactory.mirahkeys.xyz". However, without a corresponding DNS record, this host address cannot reach the load balancer.

The "mirahkeys.xyz" portion of the domain represents the configured HOSTED ZONE in AWS. To enable this functionality, it's necessary to set up the Hosted Zone in the AWS console or include it as part of your infrastructure using tools like Terraform.

Create hosted zone mirahkeys.xyz

You must have purchased a domain name from a domain provider and configured the nameservers.

![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/be0c0223-512d-46e6-b373-9ce3e184ac04)
























































































































































































































































































































































































































