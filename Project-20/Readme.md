# IMPLEMENTING CICD PIPELINE FOR TERRAFORM USING JENKINS

# Prerequisites

- Install "Docker Desktop" and start the engine

- Vscode or any IDE

# Setting Up The Environment

Let's start the project by setting up a jenkins server running in a docker container.

We will create a Dockerfile to define the configuration for our Jenkins server. This Dockerfile will include the necessary dependencies and configurations to run Jenkins seamlessly, and also to run terraform cli.

# Dockerfile for Jenkins

Jenkins comes with a docker image that can be used out of the box to run a container with all the relevant dependencies for jeenkins. But because we have unique requirement to run terraform, we need to find a way to extend the readily available jenkins image

Lets go through that quickly:

1. Create a directory and name it terraform-with-cicd

2. Create a file and name it Dockerfile

3. Add the below content in the Dockerfile

 FROM jenkins/jenkins:lts

 USER root

 RUN apt-get update && apt-get install -y \
     git \
     unzip \
     wget \
     software-properties-common \
     && rm -rf /var/lib/apt/lists/*

 RUN apt-get update && apt-get install -y gnupg software-properties-common wget \
     && wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /usr/share/keyrings/hashicorp-archive-keyring.gpg \
     && gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint \
     && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
     && apt-get update && apt-get install -y terraform \
     && rm -rf /var/lib/apt/lists/*

 WORKDIR /app

 RUN terraform --version

 USER jenkins

![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c655bb6a-baa8-4569-9aba-2d2f7362092a)

# Explaining The Dockerfile

1. # Use the Official Jenkins Base Image

   FROM jenkins/jenkins:lts

This line specifies the base image for your Dockerfile . in this case , it's using the Jenkins official Long Term Support (LTS)

2. # Switch to the root user to add additional packages

     USER root

   Thus command switches to the root user within the docker image. This is done to perform actions that require elevated permissions such as installing packages

3. # Install necessary tools and dependencies (e.g Git, Unzip, Wget)

    RUN apt-get update && apt-get install -y \
    git \
    unzip \
    wget \
    software-properties-common \
    && rm -rf /var/lib/apt/lists/*

This sections installs various tools and dependencies needed for the image. 

4. # Install Terraform

   RUN apt-get update && apt-get install -y gnupg software-properties-common wget \
    && wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | tee /usr/share/keyrings/hashicorp-archive-keyring.gpg \
    && gpg --no-default-keyring --keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg --fingerprint \
    && echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/hashicorp.list \
    && apt-get update && apt-get install -y terraform \
    && rm -rf /var/lib/apt/lists/*

This step installs Terraform. it follows step as before: updating the package list, installing dependencies

5. # Set the working directory

   WORKDIR /app

This line sets the working directory for subsequent commands to /app. This is where you will be when you enter the container

6. # Prints Terraform version to verify installation

   RUN terraform --version

This command prints the version of Terraform to the console, allowing you to verify that the installation is successful

7. # Switch back to the jenkins user

   USER jenkins

Finally, this line switches back to the jenkins user, returning to a lower priviledge level.



# Building and running the docker image

Next is to build the docker image and run it for further configuration

Make sure that you are inside the folder containing the Dockerfile. This is generally referred to as the Docker build context. The build context is the set of files located in the specified directory or path when you build a Docker image using the docker build command . The content of the build context is sent to the Docker daemon during the build process.

1. Build the custom jenkins image

    docker build -t jenkins-server . 

Notice the " . " at the end. That is the docker build context, meaning the current directory where the Dockerfile is.

![Snipe 35](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d8f5bd2e-25ec-43c8-b5c7-eee57c128dcf)

2. Run the image into a docker container

   docker run -d -p 8080:8080 --name jenkins-server jenkins-server

   This should output a hash data like:

cdbab5b31b8fb59377fa0203826aeb834bc401a01aa24252a3194f4846deb6d9

![Snipe 36](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4f0fc512-f621-4718-9f38-dcddfb29081f)


# Lets break down that command for better understanding

docker run: This is the command used to run a docker image

-d: This flag stands for "detach." It runs the container in the background, allowing you to continue using the terminal for other commands

-p 8080:8080 : This flag maps the port 8080 from the host to the port 8080 inside the container. It means that you can access the application running inside the container on your host machine's port 8080.

--name jenkins-server: This flag assigns a name to the container. In this case the name is set to "Jenkins-server"

Jenkins-server: This is the name of the Docker image that you want to run. It refers to the image named "Jenkins-server"


3. check that the container is running : docker ps

    CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                               NAMES
  cdbab5b31b8f jenkins-server   "/usr/bin/tini -- /u…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp, 50000/tcp   jenkins-server

![Snipe 37](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/062a76b0-f876-4170-af0e-55bd2d269b60)

4. Access the Jenkins server from the web browser on localhost:8080

   ![Snipe 38](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8e7fdb4d-7a12-42fb-9f0d-877516b47ce8)

5. Access the Jenkins server directly inside the container

   docker exec -it  cdbab5b31b8f   /bin/bash

![Snipe 39](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0de965aa-d691-41a9-b118-57a7b402d661)

Lets break the down the command:

docker exec: This command is used to execute a command in a running Docker container

-it: These flags are often used together
    -  -i stands for "interactive" , which allows you to interact with the container
    -  -t allocates a pseudo -TTY, or terminal enabling a more interactive session

 cdbab5b31b8f: This is the Container ID or Container Name. It uniquely identifies the running container 

 /bin/bash: This is the command you want to execute inside the container
 

6. Retrieve the initial jenkins admin password

   jenkins@cdbab5b31b8f:/app$ cat /var/jenkins_home/secrets/initialAdminPassword
a845868b3a404f39b48b1b05137b4888

   ![Snipe 40](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0485e8d6-4e2b-4c58-8e07-8be42fb8710f)

7. Installing jenkins plugins

   - paste the initial password in the web
   - click "install suggested plugins"
  
8. create admin user and access jenkins

   # Setting up Jenkins for Terraform CI/CD

   1. Fork the directory below into your own github account
  
   https://github.com/darey-devops/terraform-aws-pipeline.git

   2. Do the following to test that the code can create existing resources:
  
      A. The Provider.tf file has an S3 backend configuration. You will need to create your own bucket and update the code
      - Create an S3 bucket in your AWS account
      - update the bucket naame from the backend configuration
      B. Push your latest changes to github

      C. Run Terraform init, plan and apply and confirm everything works fine

       ![Snipe 51](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/93cf8e18-e455-411b-92dd-a7bdb2d7a92b)

       ![Snipe 52](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/97807bdd-f4cb-44eb-acf5-13a18ac6da85)

       ![Snipe 53](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5137c5f0-60a8-4ddf-8191-9fc3b1e514a1)

       ![Snipe 54](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a5ff8228-b3ef-4945-ae8c-2086f36aa27b)

       ![Snipe 55](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e386b53f-b774-4fd8-8cb3-8a435078106b)

        ![Snipe 56](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6c13bc58-91d1-47c6-b588-237ef3f4bc32)

        ![Snipe 57](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8e14d0ad-57e5-4d89-82f0-6cbc12847fd2)

       # Connect The Github Repository to Jenkins

       1. Install Jenkins Github Plugin:
     
          - Navigate to Manage Jenkins go to Plugins after that click on Available plugins and search for Github Integration
          - install the Github Integration
         
          ![Snipe 45](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f0e226b9-35cd-4a4e-9a40-ddc8bcbec4c1)

       2. Install the plugins below as you did above 

          Terraform plugins

          AWS Credentials Plugin

          ![Snipe 48](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dc08cd7e-3029-4dfe-bd90-368c7127e503)

          ![Snipe 49](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8603967c-c558-400c-acfc-1dff32918b86)

          # Configure Github Credentials in Github

          - In Github, navigate to your profile, Click on settings, Click on Developer settings
          - Generate an access token
          - copy the access token and save in a notepad
          - In jenkins, navigate to Manage Jenkins, Click on credentials
          - Click on global, select Add credentials
          - select username and password. use the Access token generated earlier as your password and specify anything descriptive as your I.D
          - in the credentials section, you will be able to see the created credential
          - Create a second credential for AWS secret and access key. if you have installed the AWS credential plugin you will see the AWS credentials as shown below . simply add the AWS secret and access key generated from AWS console. 
           
          1. Set up a Jenkins Multibranch Pipeline
         
             - From the Jenkins dashboard, click on 'New Item'
             - Give it a name and description and select Multibranch Pipeline
             - select the type of source code and the jenkinsfile
             - select the credentials to be used to connect to Github from jenkins
             - Add the repository url you forked
             - leave eveything at default and hit save
             - you will immediately see the scanning of the repository for branches and the jenkinsfile

     # Blockers

   ![Block1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ca79a4ea-f849-4ad2-bf88-bc14b9e2afcc)

   ![Block2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/25d00a65-8ce6-4168-9bcf-58be89b64a2d)

   # Solution

   The error message "Scripts not permitted to use method org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper getRawBuild" typically indicates that you are trying to access a method or property in your Jenkins pipeline script that is not allowed due to script security restrictions. Jenkins has built-in security mechanisms to prevent certain operations that could be potentially unsafe.

To resolve this issue, you have a few options:

Approved Methods: If the method you are trying to use is considered safe, you can approve it in the Jenkins script security settings. Navigate to "Manage Jenkins" > "In-process Script Approval" and look for the method that is being blocked. You can approve it from there.

Script Security Configuration: If you have administrative access to Jenkins, you can configure Script Security to allow specific methods. Go to "Manage Jenkins" > "Configure Global Security" > "Script Security" and add the necessary method signatures to the "Method Permissions" section.

Here's an example of how you might approve a method using the Script Security interface:

Go to "Manage Jenkins" > "In-process Script Approval." Look for the method that is being blocked (in this case, org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper getRawBuild). Approve the method by clicking on "Approve" next to it. Keep in mind that approving methods should be done with caution, and you should only approve methods that you trust.

![Block3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2d88529f-b960-41ef-bdbf-043fab66ddef)

![Block4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6c518d25-f703-4e4d-8aa6-b21067296780)

![Block5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/74253816-4f52-4ae5-8272-8f3be9f181cf)
 
   # Lets Talk About The Jenkinsfile

         # Pipeline 

           Pipeline {...}: This is THE Wrapper for the entire pipeline script. Everything that defines what the pipeline does is included within these braces

         # Agent

          agent any: This line specifies that the pipeline can run on any available agent

         # Environment

           environment {....}: This section is used to define environment variables that are applicable to all stages of the pipeline.

           TF_CLI_ARGS= '-no-color': sets an environment variable for Terraform. it tells Terraform to not colorize the output which can be used for logging readability in CI/CD environments.

      # Stages

           stages {...}: This block defines the various stages of the pipeline

            stage('checkout') {...}: This is the first stage, named 'checkout'. it's typically used to check out source code from a version control system

            checkout scm: checks out source code from the Source Control Management (SCM) system configured for the job

      # Stage: Terraform Plan

       stage('Terraform Plan') {...}: This stage is responsible for running a Terraform plan

       withCredentials([aws(...)]) {...}: This block securely injects AWS credentials into the environment

        sh 'Terraform init': initializes Terraform in the current directory

        sh 'Terraform plan -out=tfplan': Runs Terraform plan and output the plan to a file named 'Tfplan'

      # Stage: Terraform Apply

      stage('Terraform Apply') {...}: This stage applies the changes from Terraform plan to make infrastructure changes

      when {...}:This block sets conditions for executing this stage. expression{env.BRANCH_NAME == 'main'}: This conditions check if the pipeline is running on the main branch

      expression {...}: checks if the build was triggered by a user action

      input message: 'Do you want to apply changes?', ok: 'Yes' : A manual intervention step asking for confirmation before proceeding. if yes is clicked, it runs the terraform init and apply otherwise the pipeline is aborted.
          

















          

















          








   


























