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

3. check that the container is running : docker ps

    CONTAINER ID   IMAGE            COMMAND                  CREATED              STATUS              PORTS                               NAMES
  cdbab5b31b8f jenkins-server   "/usr/bin/tini -- /u…"   About a minute ago   Up About a minute   0.0.0.0:8080->8080/tcp, 50000/tcp   jenkins-server

![Snipe 37](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/062a76b0-f876-4170-af0e-55bd2d269b60)

4. Access the Jenkins server from the web browser on localhost:8080

   ![Snipe 38](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8e7fdb4d-7a12-42fb-9f0d-877516b47ce8)

5. Access the Jenkins server directly inside the container

   docker exec -it  cdbab5b31b8f   /bin/bash






















   


























