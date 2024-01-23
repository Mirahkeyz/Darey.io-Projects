# Container Migration Project From VM(EC2) To Containers

Install Docker and prepare for migration to the Cloud First, we need to install Docker Engine, which is a client-server application that contains:

A server with a long-running daemon process dockerd. APIs that specify interfaces that programs can use to talk to and instruct the Docker daemon. A command-line interface (CLI) client docker. You can learn how to install Docker Engine on your PC here

Before we proceed further, let us understand why we even need to move from VM to Docker.

As you have already learned – unlike a VM, Docker allocated not the whole guest OS for your application, but only isolated minimal part of it – this isolated container has all that your application needs and at the same time is lighter, faster, and can be shipped as a Docker image to multiple physical or virtual environments, as long as this environment can run Docker engine. This approach also solves the environment incompatibility issue. It is a well-known problem when a developer sends his application to you, you try to deploy it, deployment fails, and the developer replies, "- It works on my machine!". With Docker – if the application is shipped as a container, it has its own environment isolated from the rest of the world, and it will always work the same way on any server that has Docker engine.

As a part of this project, you will use already well-known by you Jenkins for Continous Integration (CI). So, when it is time to write Jenkinsfile, update your Terraform code to spin up an EC2 instance for Jenkins and run Ansible to install & configure it.

To begin our migration project from VM based workload, we need to implement a Proof of Concept (POC). In many cases, it is good to start with a small-scale project with minimal functionality to prove that technology can fulfill specific requirements. So, this project will be a precursor before you can move on to deploy enterprise-grade microservice solutions with Docker.

You can start with your own workstation or spin up an EC2 instance to install Docker engine that will host your Docker containers.

Remember our Tooling website? It is a PHP-based web solution backed by a MySQL database – all technologies you are already familiar with and which you shall be comfortable using by now.

So, let us migrate the Tooling Web Application from a VM-based solution into a containerized one.

MySQL in container Let us start assembling our application from the Database layer – we will use a pre-built MySQL database container, configure it, and make sure it is ready

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8dc875eb-0c69-4d06-937c-5e287d688ccf)

# Step 1: Pull MySQL Docker Image from Docker Hub Registry

docker pull mysql/mysql-server:latest

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/abb12cab-7d95-4f84-938b-d538765c8a9a)

NB: To run Docker commands without having to type sudo every time, you can add your user to the docker group. sudo usermod -aG docker $USER Replace $USER with your actual user. Log out and log back in for the changes to take effect.

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7fd93e33-8bff-4dd6-8857-2d9d42701691)

List the images to check that you have downloaded them successfully:

docker image ls

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/481564ea-ba13-477b-9ac5-42c0627b77e5)

# Step 2: Deploy the MySQL Container to your Docker Engine

docker run --name <container_name> -e MYSQL_ROOT_PASSWORD=<my-secret-pw> -d <name_of_image>

- Replace <container_name> with the name of your choice. If you do not provide a name, Docker will generate a random one.
- The -d option instructs Docker to run the container as a service in the background
- Replace with your chosen password
- In the command above, we used the latest version tag. This tag may differ according to the image you downloaded
  
check to see if the MySQL container is running using docker ps -a

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/612403ff-145c-4728-ad0f-a9d022908661)

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d1d73578-2dac-45c7-a453-d347ab706983)


# Step 3: Connecting to the MySQL Docker Container

We can either connect directly to the container running the MySQL server or use a second container as a MySQL client. Let us see what the first option looks like.

Approach 1

Connecting directly to the container running the MySQL server:
docker exec -it mysql bash

or

docker exec -it <name_of_container> <command_to_be_run>

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d2179d79-6a23-4678-86e5-244e081a6e70)

Let's break down the command:

docker exec: This is the Docker command for executing a command in a running container.

-it: These are the same flags we discussed before, used together to make the interaction with the container's command prompt interactive and allocate a pseudo-TTY (terminal).

mysql-server: This is the name or ID of the container you want to execute the command in, which is assumed to be a MySQL container.

mysql -uroot -p: This part of the command is what you're actually running inside the container. It's invoking the MySQL client and passing arguments to it:

-uroot: This specifies the MySQL user you want to connect as. In this case, you're connecting as the "root" user.

-p: This tells the MySQL client to prompt for the password.

stop and remove the previous mysql docker container and verify that the container is deleted docker ps -a docker stop mysql-server  docker rm mysql or <container ID>

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/762712d9-d9a2-4e46-99c1-3c7ed6799dd4)

# Create a network:

docker network create --subnet=172.18.0.0/24 tooling_app_network

Creating a custom network is not necessary because even if we do not create a network, Docker will use the default network for all the containers you run. By default, the network we created above is of DRIVER Bridge. So, also, it is the default network. You can verify this by running the docker network ls command.

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a5397976-fc0b-4448-9bbe-dbc6d2eec452)

# Run the MySQL Server container using the created network

First, let us create an environment variable to store the root password:

$ export MYSQL_PW= verify the environment variable is created echo $MYSQL_PW

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e2eb130b-153e-406d-b35e-b18bf2c218d7)

Then, pull the image and run the container, all in one command like below:

docker run --network tooling_network -h mysqlserverhost --name=mysql-server -e MYSQL_ROOT_PASSWORD=$MYSQL_PW -d mysql/mysql-server:latest

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b7202074-168f-4989-9448-0df79211a3e1)

docker run: This is the command to run a Docker container.

--network tooling_app_network: This specifies the network that the container should be connected to. In this case, the container will be connected to a Docker network named tooling_app_network.

-h mysqlserverhost: This option sets the hostname for the container. The hostname is specified as mysqlserverhost.

--name=mysql-server: This assigns a name to the running container, which will be "mysql-server".

-e MYSQL_ROOT_PASSWORD=$MYSQL_PW: This sets an environment variable named MYSQL_ROOT_PASSWORD inside the container. The value of this variable is determined by the shell environment variable $MYSQL_PW.

-d mysql/mysql-server:latest: This specifies the Docker image to use for creating the container. In this case, it pulls the latest version of the mysql/mysql-server image from Docker Hub and runs it in detached mode (-d), meaning the container will run in the background.

Create a file and name it create_user.sql and add the below code in the file:

CREATE USER ''@'%' IDENTIFIED BY ''; GRANT ALL PRIVILEGES ON * . * TO ''@'%';

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b696d6b0-b6f3-4b2c-9835-96a23897e3f2)

Run the script:

Ensure you are in the directory create_user.sql file is located or declare a path

docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < create_user.sql

# Connecting to the MySQL server from a second container running the MySQL client utility

The good thing about this approach is that you do not have to install any client tool on your laptop, and you do not need to connect directly to the container running the MySQL server.

Run the MySQL Client Container:

docker run --network tooling_network --name mysql-client -it --rm mysql mysql -h mysqlserverhost -u -p

docker run: This is the command to run a Docker container.

--network tooling_app_network: This flag specifies the network to which the container should be connected. In this case, the container will be connected to a network named tooling_app_network.

--name mysql-client: This flag assigns a name to the running container. The name of the container will be set to mysql-client.

-it: These flags combine to allocate a pseudo-TTY and keep the input/output interaction with the container alive. This is typically used when you want to interact with the container's shell or command-line interface.

--rm: This flag tells Docker to remove the container automatically after it exits. This is useful for temporary containers like the MySQL client, which is used for a specific task and then discarded.

mysql: This is the name of the Docker image that will be used to create the container. It refers to an image that contains the MySQL client tools.

mysql -h mysqlserverhost -u -p: This is the command that will be executed inside the container. It runs the MySQL client and provides the following options:

-h mysqlserverhost: This option specifies the hostname (or IP address) of the MySQL server to connect to. -u: This option is used to provide the MySQL username for authentication. However, there seems to be a missing value after -u, which should be replaced with the actual username you want to use. -p: This option prompts you to enter the MySQL password for authentication. After entering this option, the MySQL client will wait for you to input the password interactively.

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e17ac9d4-9e16-4627-b85f-c29676759359)

# Prepare database schema

Now you need to prepare a database schema so that the Tooling application can connect to it.

git clone https://github.com/darey-devops/tooling.git

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/91d6d537-64cf-4f12-91c1-d67708ab8b4f)

On your terminal, export the location of the SQL file

export tooling_db_schema=/tooling_db_schema.sql

You can find the tooling_db_schema.sql in the tooling/html/tooling_db_schema.sql folder of cloned repo.

Verify that the path is exported

echo $tooling_db_schema

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d26055b4-6036-4c15-82fe-42ddbf31ec1d)

Use the SQL script to create the database and prepare the schema. With the docker exec command

docker exec -i mysql-server mysql -uroot -p$MYSQL_PW < $tooling_db_schema

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0ccfaf41-477d-4020-a574-2e5a85c87cbe)

Update the .env file with connection details to the database

The .env file is located in the html tooling/html/.env folder but not visible in terminal.

sudo vi .env

MYSQL_IP=mysqlserverhost

MYSQL_USER=username

MYSQL_PASS=client-secrete-password

MYSQL_DBNAME=toolingdb

Ensure you are inside the directory "tooling" that has the file Dockerfile and build your container :

docker build -t tooling:0.0.1 .

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c25b40cd-a89e-42f5-bdc3-3ed92ccd3b57)

Run the container: docker run --network tooling_app_network -p 8085:80 -it tooling:0.0.1

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/09f0df59-d497-4af1-b5a0-3c938ea8a557)

# MAJOR CHALLENGES

Docker image doesn't build with the specified ENV

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/82af4a29-bc0f-4a7c-b8f9-77da62a561b4)

# FIX

cd tooling/html

sudo vi db_conn.php

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ba697d0e-8c8e-444c-89a9-e54521d7ca21)

If everything works, you can open the browser and type http://publicIP:8085 You will see the login page.

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/919df9b5-62bb-4b18-b71b-90ed66edc4d9)

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3ea76072-3655-4233-86f4-4c0bef59c341)


# PRACTICE TASK

Implement a POC to migrate the PHP-Todo app into a containerized application.

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8c0123db-df43-4fb7-b398-1aedd43d9623)

# Part 1

Write a Dockerfile for the TODO app

Run both database and app on your laptop Docker Engine Access the application from the browser


























































































to receive requests from our PHP application.
