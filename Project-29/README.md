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

```
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

![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/829d3f7d-b186-436d-930f-e390f2c1af34)

Note:

CLASS - The nginx controller class name nginx

HOSTS - The hostname to be used in the browser tooling.artifactory.mirahkeys.xyz

ADDRESS - The loadbalancer address that was created by the ingress controller

Configure DNS

When accessing the tool, sharing the lengthy load balancer address poses significant inconvenience. The ideal solution involves creating a DNS record that's easily readable by humans and capable of directing requests to the balancer. This exact configuration is set within the ingress object as host: "tooling.artifactory.mirahkeys.xyz". However, without a corresponding DNS record, this host address cannot reach the load balancer.

The "mirahkeys.xyz" portion of the domain represents the configured HOSTED ZONE in AWS. To enable this functionality, it's necessary to set up the Hosted Zone in the AWS console or include it as part of your infrastructure using tools like Terraform.

Create hosted zone mirahkeys.xyz

You must have purchased a domain name from a domain provider and configured the nameservers.

![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/adb10350-63d8-406e-9d6b-e4f62e71f105)

Ensure that you utilize the Nameservers specified in the hosted zone to set up the Nameservers within your DNS provider, such as GoDaddy, Namecheap, and others.

![Snipe 35](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/47143748-b9c4-4204-9398-ba1c4efbb070)

Create Route53 record

To establish a Route53 record, navigate to the hosted zone where essential DNS records are managed. For Artifactory, let's configure a record directing to the load balancer of the ingress controller. You have two choices: utilize either the CNAME or AWS Alias method.

If opting for the CNAME Method,

Choose the desired HOSTED ZONE.
Click on the "Create Record" button to proceed.

Please verify the DNS record's successful propagation. Go to DNS checker and choose CNAME to check the record. Make sure there are green ticks next to each location on the left-hand side. Please note that it may take some time for the changes to propagate.

![Snipe 36](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3b9d5f47-ff54-4d52-832d-f8918cbfe5ee)

We can also check this using the command

$ nslookup -type=ns tooling.artifactory.mirahkeys.xyz

![Snipe 37](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0827367d-cda8-40d9-b819-01fa742123c6)

AWS Alias Method

In the create record section, type in the record name, and toggle the alias button to enable an alias. An alias is of A DNS record type which basically routes directly to the load balancer. In the choose endpoint bar, select Alias to Application and Classic Load Balancer.

Accessing the application from the browser

Accessing the application through your browser Presently, our Kubernetes-hosted application is reachable from outside sources. When you visit your domain's specific URL - tooling.artifactory.mirahkeys.xyz, the artifactory application should be accessible.

Accessing the application using the HTTP protocol

![Snipe 38](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d9d42ee8-2da7-4749-87d2-6057781e70a2)

When accessing the application via the HTTPS protocol in Chrome, you might see a message stating that the site is reachable but insecure. This happens when the site doesn't have a trusted TLS/SSL certificate, or it lacks one entirely.

To view the certificate, click on the "Not Secure" section and then select "Certificate Not Valid."

Explore Artifactory Web UI

Get the default username and password - Run a helm command to output the same message after the initial install

$ helm test artifactory -n tools

![Snipe 39](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/006b81dc-c2e3-4605-b2ad-8e30f5adbb22)

Insert the username and password to load the Get Started page

![Snipe 40](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7cca1784-94f5-4849-a389-706b8a1378d7)

Reset the admin password

![Snipe 41](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6fac2c4a-b405-4a81-800f-813338067649)

Activate the Artifactory License. You will need to purchase a license to use Artifactory enterprise features.

For learning purposes, you can apply for a free trial license. Simply fill the form here and a license key will be delivered to your email in few minutes.

N/B: Make sur to check the box "schedule a technical demo"

Check your email

![Snipe 42](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7fabae3b-72bf-4f09-b423-1f7ef806dc4b)

Copy and paste in the license section.

Set Base URL. Be sure to use HTTPS i.e https://tooling.artifactory.mirahkeys.xyz

![Snipe 43](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6d4214b8-f590-4af7-9d9b-14f9cfed7c24)

Click on next

Skip Proxy settings and creating the repo for now.

![Snipe 44](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/79d202d0-0328-4fc9-8513-962949bdb2fd)

![Snipe 45](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/471fcb98-48ce-4684-bac6-cad2c40f7ebf)

Finish the setup

![Snipe 46](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/786e7a74-2cff-452c-8d11-a76aa7e220bb)

![Snipe 47](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0dd9fe9e-b7d1-4cb9-b6a9-513386c14c76)

Next, its time to fix the TLS/SSL configuration so that we will have a trusted HTTPS URL

# Deploying Cert-Manager and managing TLS/SSL for Ingress

Transport Layer Security (TLS), the successor of the now-deprecated Secure Sockets Layer (SSL), is a cryptographic protocol designed to provide communications security over a computer network. The TLS protocol aims primarily to provide cryptography, including privacy (confidentiality), integrity, and authenticity through the use of certificates, between two or more communicating computer applications. The certificates required to implement TLS must be issued by a trusted Certificate Authority (CA). To see the list of trusted root Certification Authorities (CA) and their certificates used by Google Chrome, you need to use the Certificate Manager built inside Google Chrome as shown below:

Open the settings section of google chrome

Search for security and click on Security - Safe Browsing (protection from dangerous sites) and other security settings

Select Manage Certificates

![Snipe 48](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9cf24eea-d300-4177-8306-23d8949a6c41)

View the installed certificates in your browser

![Snipe 49](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/62a3094f-8523-4ca6-bda0-1ea666b5b10b)

Certificate Management in Kubernetes

Streamlining the acquisition and management of trusted certificates from dynamic certificate authorities is a challenging task. It involves overseeing certificate requests, issuance, expiration tracking, and application-specific certificate management, which can incur significant administrative overhead. This often requires the creation of intricate scripts or programs to handle these complexities.

Cert-Manager is a lifesaver in simplifying these processes. Within Kubernetes clusters, Cert-Manager introduces certificates and certificate issuers as resource types. It streamlines the acquisition, renewal, and utilization of certificates same approach the Ingress Controllers facilitate the creation of Ingress resources within the cluster.

Cert-Manager empowers administrators by enabling the creation of certificate resources and additional resources essential for seamless certificate management. It supports certificate issuance from various sources like Let's Encrypt, HashiCorp Vault, Venafi, and private PKIs. The resulting certificates are stored as Kubernetes secrets, housing both the private key and the public certificate for easy access and utilization.

In this Project, we will use Let's Encrypt with cert-manager.

The certificates issued by Let's Encrypt will work with most browsers because the root certificate that validates all it's certificates is called “ISRG Root X1” which is already trusted by most browsers and servers. You will find ISRG Root X1 in the list of certificates already installed in your browser.

![Snipe 50](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1477b83a-b359-4bb4-84c2-94561eb40786)

Cert-maanager will ensure certificates are valid and up to date, and attempt to renew certificates at a configured time before expiry.

Cert-Manager high Level Architecture

Cert-manager works by having administrators create a resource in kubernetes called certificate issuer which will be configured to work with supported sources of certificates. This issuer can either be scoped globally in the cluster or only local to the namespace it is deployed to. Whenever it is time to create a certificate for a specific host or website address, the process follows the pattern seen in the image below.

# Deploying Cert-manager

Lets Deploy cert-manager helm chart in Artifact Hub, follow the installation guide and deploy into Kubernetes

Create a namespace cert-manager

$ kubectl create ns cert-manager

Before installing the chart, you must first install the cert-manager CustomResourceDefinition resources. This is performed in a separate step to allow you to easily uninstall and reinstall cert-manager without deleting your installed custom resources.

$ kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.13.2/cert-manager.crds.yaml -n cert-manager

Add the Jetstack helm repo

$ helm repo add jetstack https://charts.jetstack.io

Install the cert-manager helm chart

$ helm install cert-manager jetstack/cert-manager --version v1.13.2 --namespace cert-manager

![Snipe 51](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f720631b-f8f2-4220-bcae-2af0faf0713a)

Certificate Issuer

In order to begin issuing certificates, you will need to set up a ClusterIssuer or Issuer resource.

Create an Issuer. We will use a Cluster Issuer so that it can be scoped globally. Assuming that we will be using dybran.com domain. Simply update this yaml file and deploy with kubectl. In the section that follows, we will break down each part of the file.

```
cat <<EOF > cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  namespace: "cert-manager"
  name: "letsencrypt-prod"
spec:
  acme:
    server: "https://acme-v02.api.letsencrypt.org/directory"
    email: "infradev@oldcowboyshop.com"
    privateKeySecretRef:
      name: "letsencrypt-prod"
    solvers:
    - selector:
        dnsZones:
          - "mirahkeys.xyz"
      dns01:
        route53:
          region: "us-west-1"
          hostedZoneID: "Z02607852KCM9RZXQXSMI"
EOF

```

The initial section shows the Kubernetes configuration, specifying the apiVersion, Kind and metadata. In this context, the Kind refers to a ClusterIssuer, indicating its global scope.

In the spec section, an ACME - Automated Certificate Management Environment issuer type is specified here. When you create a new ACME Issuer, cert-manager will generate a private key which is used to identify you with the ACME server. Certificates issued by public ACME servers are typically trusted by client's computers by default. This means that, for example, visiting a website that is backed by an ACME certificate issued for that URL, will be trusted by default by most client's web browsers. ACME certificates are typically free. Let’s Encrypt uses the ACME protocol to verify that you control a given domain name and to issue you a certificate. You can either use the let's encrypt Production server address https://acme-v02.api.letsencrypt.org/directory which can be used for all production websites. Or it can be replaced with the staging URL https://acme-staging-v02.api.letsencrypt.org/directory for all Non-Production sites.

The privateKeySecretRef has configuration for the private key name you prefer to use to store the ACME account private key. This can be anything you specify, for example letsencrypt-prod.

This section is part of the spec that configures solvers which determines the domain address that the issued certificate will be registered with. dns01 is one of the different challenges that cert-manager uses to verify domain ownership. Read more on DNS01 Challenge here. With the DNS01 configuration, you will need to specify the Route53 DNS Hosted Zone ID and region. Since we are using EKS in AWS, the IAM permission of the worker nodes will be used to access Route53. Therefore if appropriate permissions is not set for EKS worker nodes, it is possible that certificate challenge with Route53 will fail, hence certificates will not get issued.

The next section under the spec that configures solvers which determines the domain address that the issued certificate will be registered with. dns01 is one of the different challenges that cert-manager uses to verify domain ownership. Read more on DNS01 Challenge here. With the __ DNS01__ configuration, you will need to specify the Route53 DNS Hosted Zone ID and region. Since we are using EKS in AWS, the IAM permission of the worker nodes will be used to access Route53. Therefore if appropriate permissions is not set for EKS worker nodes, it is possible that certificate challenge with Route53 will fail, hence certificates will not get issued. The other possible option is the HTTP01 challenge, but we won't be using that.

To get the Hosted Zone ID

$ aws route53 list-hosted-zones

![Snipe 52](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2641446b-2c27-4ecd-8e33-f13860ed14c5)

Update the yaml file.

```
cat <<EOF > cluster-issuer.yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  namespace: "cert-manager"
  name: "letsencrypt-prod"
spec:
  acme:
    server: "https://acme-v02.api.letsencrypt.org/directory"
    email: "infradev@oldcowboyshop.com"
    privateKeySecretRef:
      name: "letsencrypt-prod"
    solvers:
    - selector:
        dnsZones:
          - "mirahkeys.xyz"
      dns01:
        route53:
          region: "us-west-1"
          hostedZoneID: "Z02607852KCM9RZXQXSMI"
EOF

```

Deploy with kubectl in the cert-manager namespace

$ kubectl apply -f cluster-issuer.yaml -n cert-manager

![Snipe 53](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/98eb1045-50d5-422c-8374-e737f13b4ad7)

$ kubectl get pods -n cert-manager

![Snipe 54](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0a3420bc-beac-4252-823c-e8ff65df715c)

With the ClusterIssuer properly configured, it is now time to start getting certificates issued.

Configuring Ingress for TLS

To ensure that every created ingress also has TLS configured, we will need to update the ingress manifest with TLS specific configurations.

```
cat <<EOF > artifactory-ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: nginx
  name: artifactory
spec:
  rules:
  - host: "tooling.artifactory.mirahkeys.xyz"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: artifactory-artifactory-nginx
            port:
              number: 80
  tls:
  - hosts:
    - "tooling.artifactory.mirahkeys.xyz"
    secretName: "tooling.artifactory.mirahkeys.xyz"
EOF

```

Create the updated artifactory-ingress.yaml

$ kubectl apply -f artifactory-nginx.yaml -n tools

![Snipe 55](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/06c0f910-848c-4139-9e90-3c18406aa403)

The most significant updates to the ingress definition is the annotations and tls sections.

Annotations are used similar to labels in kubernetes. They are ways to attach metadata to objects.

Difference between Annotations and Label

Annotations and labels serve distinct roles in Kubernetes resource management.

Labels function as identifiers for resource grouping when used alongside selectors. To ensure efficient querying, labels adhere to RFC 1123 constraints, limiting their length to 63 characters. Therefore, utilizing labels is ideal for Kubernetes when organizing related resources into sets.

On the other hand, annotations cater to "non-identifying information" or metadata that Kubernetes doesn't rely on. Unlike labels, there are no constraints imposed on annotation keys and values. Consequently, if the goal is to furnish additional information for human comprehension regarding a specific resource, annotations offer a more suitable choice.

The Annotation added to the Ingress resource adds metadata to specify the issuer responsible for requesting certificates. The issuer here will be the same one we have created earlier with the name letsencrypt-prod

```
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
```

The other section is tls where the host name that will require https is specified. The secretName also holds the name of the secret that will be created which will store details of the certificate key-pair. i.e Private key and public certificate.

```
  tls:
  - hosts:
    - "tooling.artifactory.dybran.com"
    secretName: "tooling.artifactory.dybran.com"
```

commands to see each resource at each phase.

$ kubectl get certificaterequest -n tools

$ kubectl get order -n tools

$ kubectl get challenge -n tools

$ kubectl get certificate -n tools

Problem encoutered

After applying the above command, i ran into some issues, my certicate remains in pending state.

![Snipe 56](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cd45937a-422b-43cc-8911-21521f90d791)

During investigation on the challenge resource, I noticed a permission issue

$ kubectl describe challenge tooling.artifactory.mirahkeys.xyz-1-1046896647-1545754586 -n tools

![Snipe 57](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0436de21-5021-427f-b036-bfcb409cfd9c)

This means that there is an issue with presenting a challenge due to a permissions error related to Route 53 in AWS. The error indicates that the IAM role being assumed ( eksctl-miracle-eks-tooling-nodegro-NodeInstanceRole-yiGRAWmsjZxK) does not have the necessary permissions to perform the route53:ChangeResourceRecordSets action on the specified hosted zone (Z08522561JSS4FBNMMK3E).

Resolution

To resolve the permissions issue for the IAM role  eksctl-miracle-eks-tooling-nodegro-NodeInstanceRole-yiGRAWmsjZxK and allow it to perform the route53:ChangeResourceRecordSets action on the specific hosted zone (Z08522561JSS4FBNMMK3E).

Identify the IAM role assumed by your EKS cluster nodes -  eksctl-miracle-eks-tooling-nodegro-NodeInstanceRole-yiGRAWmsjZxK

![Snipe 58](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e6ce0047-4d0b-4c41-838e-5331a89e402e)

Update the IAM policy linked to this role by adding the required permissions for Route 53. Ensure that you authorize the route53:ChangeResourceRecordSets and route53:GetChange actions specifically for the designated hosted zone (arn:aws:route53:::hostedzone/Z08522561JSS4FBNMMK3).

![Snipe 59](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f99f49f4-b708-4e26-839f-a3e1207b741a)

You can see that the route53:ChangeResourceRecordSets is not included in the permission policy above.

You can also get the attached policies using this command

$ aws iam list-attached-role-policies --role-name eksctl-miracle-eks-tooling-nodegro-NodeInstanceRole-yiGRAWmsjZxK

Create the IAM policy to be added to the role policy

```
cat <<EOF > ChangeResourceRecordSets.json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement",
            "Effect": "Allow",
            "Action": [
                "route53:ChangeResourceRecordSets",
                "route53:GetChange"
            ],
            "Resource": [
                "arn:aws:route53:::hostedzone/Z08522561JSS4FBNMMK3E",
                "arn:aws:route53:::change/C0567608HJ3CPCXTDOBL"
            ]
        }
    ]
}
EOF

```

To obtain the route53:GetChange with the identifier (C03778642NCAAJ62J6XKO) in the above, navigate to the CNAME record for tooling.artifactory.mirahkeys.xyz, select the edit option, and subsequently click save without making any actual modifications. Afterward, proceed to click on view details.

![Snipe 60](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ded94639-de0b-45d8-aebd-50ffd1e71e03)

![Snipe 61](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/add02652-2087-42d3-aea2-724c9502c533)

Apply the updated IAM policy to the IAM role. You can do this through the AWS Management Console or by using the AWS CLI

aws iam create-policy --policy-name ChangeResourceRecordSets --policy-document file://ChangeResourceRecordSets.json

After running the create-policy command, the output will include the Amazon Resource Name (ARN) of the created policy.

Then attach the policy to the role

aws iam attach-role-policy --role-name eksctl-miracle-eks-tooling-nodegrou-NodeInstanceRole-SQcjbXR7eFcD --policy-arn arn:aws:iam::939895954199:policy/ChangeResourceRecordSets

![Snipe 62](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/08ebf9ac-b81f-407f-aec7-9faffc7f7f97)

On the policies, we will see the newly created policy

![Snipe 63](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f528ee8f-85da-46e4-91c3-cc0bff363575)

You can verify that the policy has been attached by running

aws iam list-attached-role-policies --role-name eksctl-dybran-eks-tooling-nodegrou-NodeInstanceRole-SQcjbXR7eFcD

![Snipe 64](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/faf1dc3a-4383-4e9a-93d6-24cb35a1621a)

Ensure the IAM role establishes the accurate trust relationship with the EKS cluster, enabling the cluster to assume the role. The policy should grant permissions for AWS services eks.amazonaws.com and ec2.amazonaws.com to assume roles via (sts:AssumeRole). This practice is frequently employed to authorize services or resources for particular actions or access to specific resources within your AWS setup.

On the role - eksctl-miracle-eks-tooling-nodegrou-NodeInstanceRole-HSWbuyw3gPhC, update the trust relationship with this

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": [
                    "eks.amazonaws.com",
                    "ec2.amazonaws.com"
                ]
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

![Snipe 65](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d241b1e5-2f4e-4613-b9ae-f459634ae36a)

![Snipe 66](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d4cee0ed-56e4-4f3d-9e90-ada41c667c2c)






















































































































































































































































































































































































































































































































































































































































