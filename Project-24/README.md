# Automate Infrastructure With Iac Using Terraform. (Terraform Cloud)

# Terraform Cloud

What Terraform Cloud is and why use it By now, you should be pretty comfortable writing Terraform code to provision Cloud infrastructure using Configuration Language (HCL). Terraform is an open-source system, that you installed and ran a Virtual Machine (VM) that you had to create, maintain and keep up to date. In Cloud world it is quite common to provide a managed version of an open-source software. Managed means that you do not have to install, configure and maintain it yourself – you just create an account and use it "as A Service".

Terraform Cloud is a managed service that provides you with Terraform CLI to provision infrastructure, either on demand or in response to various events.

By default, Terraform CLI performs operation on the server whene it is invoked, it is perfectly fine if you have a dedicated role who can launch it, but if you have a team who works with Terraform – you need a consistent remote environment with remote workflow and shared state to run Terraform commands. Migrate your .tf codes to Terraform Cloud Le us explore how we can migrate our codes to Terraform Cloud and manage our AWS infrastructure from there:

# Create a Terraform Cloud account

 create a new account, verify your email and you are ready to start Most of the features are free, but if you want to explore the difference between free and paid plans

 Create an organization Select "Start from scratch", choose a name for your organization and create it.

- Configure a workspace Before we begin to configure our workspace

We will use version control workflow as the most common and recommended way to run Terraform commands triggered from our git repository.

Create a new repository in your GitHub and call it terraform-cloud, push your Terraform codes developed in the previous projects to the repository.

Choose version control workflow and you will be promped to connect your GitHub account to your workspace – follow the prompt and add your newly created repository to the workspace.

Move on to "Configure settings", provide a description for your workspace and leave all the rest settings default, click "Create workspace".

- Configure variables Terraform Cloud supports two types of variables: environment variables and Terraform variables. Either type can be marked as sensitive, which prevents them from being displayed in the Terraform Cloud web UI and makes them write-only.

Set two environment variables: AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY, set the values that you used in Project 16. These credentials will be used to privision your AWS infrastructure by Terraform Cloud.

After you have set these 2 environment variables – yout Terraform Cloud is all set to apply the codes from GitHub and create all necessary AWS resources.

Now it is time to run our Terrafrom scripts, but in our previous project which was project 18, we talked about using Packer to build our images, and Ansible to configure the infrastructure, so for that we are going to make few changes to our our existing respository from Project 18. The files that would be Added is;

- AMI: for building packer images Ansible: for Ansible scripts to configure the infrastucture Before you proceed ensure you have the following tools installed on your local machine;

- packer

- Ansible Refer to this repository for guidiance on how to refactor your enviroment to meet the new changes above and ensure you go through the README.md file.

Run terraform plan and terraform apply from web console Switch to "Runs" tab and click on "Queue plan manualy" button. If planning has been successfull, you can proceed and confirm Apply – press "Confirm and apply", provide a comment and "Confirm plan"

Check the logs and verify that everything has run correctly. Note that Terraform Cloud has generated a unique state version that you can open and see the codes applied and the changes made since the last run.

- Test automated terraform plan By now, you have tried to launch plan and apply manually from Terraform Cloud web console. But since we have an integration with GitHub, the process can be triggered automatically. Try to change something in any of .tf files and look at "Runs" tab again – plan must be launched automatically, but to apply you still need to approve manually. Since provisioning of new Cloud resources might incur significant costs. Even though you can configure "Auto apply", it is always a good idea to verify your plan results before pushing it to apply to avoid any misconfigurations that can cause ‘bill shock’.

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f88f892a-7bc0-4185-a290-ddc164d4a9e0)

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/53316717-e624-41d1-8456-2fab84676d69)

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f4a70009-7ca2-49cc-9517-601fc3e4445d)

# Installing Packer on Linux(Ubuntu)

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -

sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"

sudo apt-get update && sudo apt-get install packer

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3aac6cb5-9403-488d-a46a-5377a07d4043)

# EXPORT ENVIRONMENT VARIABLES

# for WINDOWS 

setx AWS_ACCESS_KEY_ID "<YOUR_AWS_ACCESS_KEY_ID>"
setx AWS_SECRET_ACCESS_KEY_ID "<YOUR_AWS_SECRET_ACCESS_KEY>"

# for Linux

export AWS_ACCESS_KEY_ID="<YOUR_AWS_ACCESS_KEY_ID>"
export AWS_SECRET_ACCESS_KEY="<YOUR_AWS_SECRET_ACCESS_KEY>"

# check for existence

printenv | grep -i AWS_ACCESS

- create a file for the variables and provider in the ami_folder after the error below

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3322f1f2-3e88-41e3-a40f-574053bf4416)

packer {
  required_plugins {
    amazon = {
      version = ">= 0.0.2"
      source  = "github.com/hashicorp/amazon"
    }
  }
}

variable "region" {
  type    = string
  default = "us-east-1"
}

locals { timestamp = regex_replace(timestamp(), "[- TZ:]", "") }

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9c3332c1-3306-4905-b812-d1893313a44a)

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8c413eed-da71-499e-9c79-1d8c29a584f6)

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4e080943-54ac-40f7-bfa0-5c1dc24f21f0)

# Terraform plan

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c2344016-5f62-4366-be42-185253d9a1f1)

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d798a008-a6da-47a1-883e-64df23c68fc7)

# Terraform Apply

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d4ceec02-f509-4ef6-a58a-ca0b2d071bbe)

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a13132d3-12bc-453d-a567-355ae6fe2924)

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/eddc6c96-8d49-4cdc-a2ec-3e8e65179aa2)

# ANSIBLE

- Configure aws configure on the ansible server.
- Add the role path in the ansible.cfg file

roles_path = /home/ec2-user/ansible-for-terraform-prj19/roles

- Then export on the ansible directory
 export ANSIBLE_CONFIG=/home/ec2-user/ansible-for-terraform-prj19/ansible.cfg

- Confirm the ansible inventory file is reachable using the --graph
ansible-inventory -i inventory/aws_ec2.yml --graph 

- execute the ansible playbook
ansible-playbook -i inventory/aws_ec2.yml playbooks/site.yml

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/08bf6179-469c-455e-9b2e-37c6baf9ef2c)

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1e368a4e-b729-4560-80b2-4ca7eb07ed8b)

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/01ada627-429c-4c09-bda0-9b3cab21e920)

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a41d22b6-c45b-4ae1-84b9-f166c0693a44)

# Intial error on NGINX SERVER

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bc610556-9141-47db-b8f1-f0f4ed5da295)

# I resolved it by checking the internal load balancer url in the /etc/nginx/nginx.conf

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fb0f084e-24d4-408b-9ad4-81ebba9b2e1f)

# Deployment on Tooling and Wordpress

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d228a424-2b46-4dc6-a1c9-36ef45db2204)

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/30d4e7ef-298c-40fd-8bfc-67d34dd9400a)

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/134b8532-eff0-4233-af5b-bf6e3eb26709)

# ERROR CONNECTING TO DATABASE

![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fbb9e63f-06ae-444a-8a6d-f259dbf3fcd2)

- Edit the file using sudo vi wp-config.php

![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c1592ee6-687d-4629-9dd0-3f9eac9909b7)

![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d0779b3e-2e47-4a3b-b7e9-006fb3cbcaf8)

Implement the resource attachment for the load balancer on the terraform file for the nginx/web server

![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dfc7cebb-594f-4055-84ae-c25c9046d072)



















































































