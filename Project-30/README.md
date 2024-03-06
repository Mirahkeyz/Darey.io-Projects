# Working With Jenkins And Jfrog Artifactory

# Create A Local Repository For Docker

- Click on administration, Repository and then Add Repository
![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e290cedc-d16d-45b3-b04d-ab7e98a158fe)

- Since we are exploring Local Repositories, Select Local Repository

- Type Docker in the search box and select the Docker Icon

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c08d4a6a-163f-4098-a5e0-86f44c3f9844)

In the Repository Key box, type in the name of the repository you wish to create for exaple Tooling. so that all the docker images for tooling app can be pushed there

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4ce148be-7c82-4281-ba80-c4e7c330a667)

- Click on the create Local Repository

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5c8f802b-f4f2-4eeb-bdb9-605ecc381264)

- Now you can see that a Docker Repository for tooling has been created

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a14f212e-0b11-45cb-a655-a4049580ed38)

- Create a second Local Repository for Jenkins

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/63dba6a8-4741-431b-a698-c701f2343379)

# Lets Create A Virtual Repository

Remember, a virtual repository aggregates several repositories under a common URL. you can get artifacts from it but you cannot deploy anything to it

- Select virtual repository

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e6fe53d6-eac3-486f-9f2d-7f235bf53b52)

- Name the Virtual Repository as you deem fit, click on the create virtual repository button

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3eac2ca1-fa1a-437c-a5f1-3ed88e63591b)

- Now that the virtual repository is created, it is time add local repositories to it

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/57723f7c-8d32-47b1-8a68-396c7a2f0cb0)

- Scroll down the page to see the local repositories. This is where you select which local repository will be part of the virtual repository. Click on the double arrows to move them

![Snipe10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/46b977f9-8676-4bc5-bdae-def9bae920cc)

- Once Moved you will see them in the included items section

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/641d5622-0853-418c-8892-5bb88bc443f1)

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bfad5596-b23f-464c-a8e2-eb562ed222a7)

# Push docker images to the repository 

You can either pull and push docker images to the local repository for each application, or simply pull from the virtual repository. 

Lets get docker images from docker hub and push to our private registry. 

• First you will need to login to the docker registry. 
 
$ docker login tooling.artifactory.sandbox.svc.darey.io 

• A successful login will create a file here N/ .docker/config. j son Explore this file and see how the authentication data is stored. The section is an encoding of your username and password. You can try to decode it with base64 to see the output. 

```
{
        "auths": {
                "tooling.artifactory.sandbox.svc.darey.io": {
                        "auth": "YWRtaW46dGVzdF9wYXNzd29yZA=="
                }
        }
}
```

- Pull the Jenkins image from the docker hub

  $ docker pull jenkins/jenkins:jdk11

- Tag the images so that it can be pushed to the artifactory

  $ docker tag jenkins/jenkins:jdk11 https://tooling.artifactory.sandbox.svc.darey.io/jenkins/jenkins:jdk11

- Push the docker image to Artifactory

$ docker push tooling.artifactory.sandbox.svc.darey.io/jenkins/jenkins:jdk11

# Jenkins pipeline for Business Applications

In earlier projects, pipeline for the Tooling app was based on Ansible. This time, we are containerising the same application. Since the app will be running inside a kubernetes cluster within a Pod container, then the approach to Cl/CD will be different. 

There will be different elements to Cl/CD here. The build of the application, and the deployment into kubernetes. 

1. The Dockerft le used to build the tooling app's docker image will have its own Cl/CD pipeline
2. The helm charts used to deploy the application into kubernetes will require its own Cl/CD pipeline
   
Therefore, we will begin with the first one. But without jumping straight to creating pipelines, we must also consider the best way to automate the deployment and configuration of the Cl/CD tool (Jenkins) such that it is fit for purpose, and can be easily recreated with the exact same config. 

Self Challenge Task: 

• Using helm, deploy Jenkins and sonarqube into the tools namespace. 
• Configure TLS based ingress for both Jenkins and Sonarqube. (Use the Helm Values to configure Ingress directly) 
• Create a Multibranch pipeline for the tooling app. 
• Connect the tooling app from Github https://github.com/darey-devops/tooling 

If you were able to successfully implement that challenge, then thumbs up to you. 

Lets go through each of the steps. 

# Deploy Jenkins with helm 

Phase 1- Deploy without any custom configuration to the Helm Values (Do the whole of this part yourself) 

1. Without any custom configuration, get the Jenkins Helm chart from artifacthub.io, and deploy using the default values.

2. Configure DNS for jenkins and route traffic to the ingress controller load balancer

3. Deploy an ingress without TLS

4. Ensure that you are able to access the configured URL

5. Ensure that you are able to loon to Jenkiins. 

6. Update the Ingress and configure TLS for the URL

Phase 2 - Use an override values file to customize Jenkins deployment (You will be guided in this part) 

The default helm values file has so many configuration tweaks to customize Jenkins. We will explore some of these and ensure that all desired configuration is done using the helm values only. For example, using another YAML file to configure the ingress is removed. 

Tasks: 

1. Configure Jenkins Ingress using Helm Values

2. Automate Jenkins plugin installation

3. Automating Jenkins Configuration As Code (JCasC)

4. Automating Jenkins backups to AWS S3 

Lets get started (Note that the version numbers in provided screenshots or code snippets may have changed from the time of writing. Ensure to update the numbers as provided by the Helm chart) 

1. Configure Jenkins Ingress using Helm Values 

o Delete the previous Ingress using kubectl delete command. 

o Create a new file and name it jenkins-values-overide.yaml  

Find the below section in the default values file. Copy and paste it exactly in the jenkins-values-overide.yaml 

```
controller:
  # Used for label app.kubernetes.io/component
  componentName: "jenkins-controller"
  image: "jenkins/jenkins"
  # tag: "2.332.3-jdk11"
  tagLabel: jdk11
  imagePullPolicy: "Always"
```

• Then do a helm upgrade using the jenkins-values-overide.yaml file 

$ helm upgrade -i jenkins jenkinsci/jenkins -n tools -f jenkins-values-overide.yaml 

• This should do an upgrade, but without any specific changes to the existing deployment. Thats because no configuration change has occured. All we now have is a shortened values file that can be easily read without too many options. With this file you can now start making configuration updates. 

• To configure Jenkins ingress directly from the helm values, simply search for the ingress: section in the default values file and copy the entire section to the override values file. 

The default one should look similar to this

```
  ingress:
    enabled: false
    # Override for the default paths that map requests to the backend
    paths: []
    # - backend:
    #     serviceName: ssl-redirect
    #     servicePort: use-annotation
    # - backend:
    #     serviceName: >-
    #       {{ template "jenkins.fullname" . }}
    #     # Don't use string here, use only integer value!
    #     servicePort: 8080
    # For Kubernetes v1.14+, use 'networking.k8s.io/v1beta1'
    # For Kubernetes v1.19+, use 'networking.k8s.io/v1'
    apiVersion: "extensions/v1beta1"
    labels: {}
    annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
    # For Kubernetes >= 1.18 you should specify the ingress-controller via the field ingressClassName
    # See https://kubernetes.io/blog/2020/04/02/improvements-to-the-ingress-api-in-kubernetes-1.18/#specifying-the-class-of-an-ingress
    # ingressClassName: nginx
    # Set this path to jenkinsUriPrefix above or use annotations to rewrite path
    # path: "/jenkins"
    # configures the hostname e.g. jenkins.example.com
    hostName:
    tls:
    # - secretName: jenkins.cluster.local
    #   hosts:
    #     - jenkins.cluster.local

```

o Given the ingress section copied above, attempt to update it with values for your specific ingress. Have a look through the ingress file you used to create the ingress previously and see what sections you require to update the values. For example 
■ Enable the ingress value from false to -true 

■ Add the annotations you already used to create the ingress before 

■ Add the hostname 

■ Update the us section with the 'a-ccrcr-di e and s information. O The final output should look similar to this 

```
ingress:
  enabled: true
  apiVersion: "extensions/v1beta1"
  annotations: 
    cert-manager.io/cluster-issuer: "letsencrypt-production"
    kubernetes.io/ingress.class: nginx
  hostName: tooling.jenkins.sandbox.svc.darey.io
  tls:
  - secretName: tooling.jenkins.sandbox.svc.darey.io
    hosts:
      - tooling.jenkins.sandbox.svc.darey.io
```

o Now upgrade the Jenkins deployment with helm upgrade command. Remember to specify the override yaml file with the -f file. 


























