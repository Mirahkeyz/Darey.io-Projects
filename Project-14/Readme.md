# CONTINUOUS INTEGRATION WITH JENKINS | ANSIBLE | ARTIFACTORY | SONARQUBE | PHP

To emphasize a typical CI Pipeline further, let us explore the diagram below.

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/046c03a4-9fe9-4146-aeeb-aa02b68d0838)

- Version Control: This is the stage where developers’ code gets committed and pushed after they have tested their work locally.
  
- Build: Depending on the type of language or technology used, we may need to build the codes into binary executable files (in case of compiled languages) or just package the codes together with all necessary dependencies into a deployable package (in case of interpreted languages).
  
- Unit Test: Unit tests that have been developed by the developers are tested. Depending on how the CI job is configured, the entire pipeline may fail if part of the tests fails, and developers will have to fix this failure before starting the pipeline again. A Job by the way, is a phase in the pipeline. Unit Test is a phase, therefore it can be considered a job on its own.
  
- Deploy: Once the tests are passed, the next phase is to deploy the compiled or packaged code into an artifact repository. This is where all the various versions of code including the latest will be stored. The CI tool will have to pick up the code from this location to proceed with the remaining parts of the pipeline.
  
- Auto Test: Apart from Unit testing, there are many other kinds of tests that are required to analyse the quality of code and determine how vulnerable the software will be to external or internal attacks. These tests must be automated, and there can be multiple environments created to fulfil different test requirements. For example, a server dedicated for Integration Testing will have the code deployed there to conduct integration tests. Once that passes, there can be other sub-layers in the testing phase in which the code will be deployed to, so as to conduct further tests. Such are User Acceptance Testing (UAT), and another can be Penetration Testing. These servers will be named according to what they have been designed to do in those environments. A UAT server is generally be used for UAT, SIT server is for Systems Integration Testing, PEN Server is for Penetration Testing and they can be named whatever the naming style or convention in which the team is used. An environment does not necessarily have to reside on one single server. In most cases it might be a stack as you have defined in your Ansible Inventory. All the servers in the inventory/dev are considered as Dev Environment. The same goes for inventory/stage (Staging Environment) inventory/preprod (Pre-production environment), inventory/prod (Production environment), etc. So, it is all down to naming convention as agreed and used company or team wide.
  
- Deploy to production: Once all the tests have been conducted and either the release manager or whoever has the authority to authorize the release to the production server is happy, he gives green light to hit the deploy button to ship the release to production environment. This is an Ideal Continuous Delivery Pipeline. If the entire pipeline was automated and no human is required to manually give the Go decision, then this would be considered as Continuous Deployment. Because the cycle will be repeated, and every time there is a code commit and push, it causes the pipeline to trigger, and the loop continues over and over again.
  
- Measure and Validate: This is where live users are interacting with the application and feedback is being collected for further improvements and bug fixes.
  
In this project, I will be setting up a CI/CD Pipeline for a PHP based application.

This project is architected in two major repositories with each repository containing its own CI/CD pipeline written in a Jenkinsfile

- ansible-config-mgt Repository: This repository employs the use of Jenkinsfile to configure infrastructure required to carry out processes for our application to run using ansible roles.

- PHP-todo Repository: This repository uses Jenkinsfile to run the processes needed to build the PHP application. These processes include testing, building, packaging and deploy.

For this project, i will be using RHEL 8. The tools we will be using to build, test, run code analysis, package and deploy our PHP application are Github, jenkins, Sonarqube and jfrog artifactory.

# Configuring Ansible For Jenkins Deployment

- First, we start our Jenkins instance.

  ![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/50815d1f-42d4-4037-b9ef-7ba239457369)

- Connect to our vscode

  Host Jenkins-ansible
           Hostname  ( the Public IPv4 Dns link)
           User ec2-user
           IdentityFile /Users/thinkpad/downloads/jenkins-key.pem   (the location were your key-pair is)

  ![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7299186c-5015-4998-8bd1-48110c95b968)

  - Spin up a vscode terminal
 
  - If it is a new jenkins-ansible server install ansible and others with the below commands
 
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum install -y ansible-2.9.25
sudo yum install python3 python3-pip wget unzip git -y
sudo python3 -m pip install --upgrade setuptools
sudo python3 -m pip install --upgrade pip
sudo python3 -m pip install PyMySQL
python3 -m pip install mysql-connector-python
python3 -m pip install psycopg2==2.7.5 --ignore-installed
ansible-galaxy collection install community.postgresql
ansible-galaxy collection install community.mysql
 
  - Go to jenkins.io and follow the Fedora long term support release process to install jenkins on the vscode terminal

  - After installing go to ur instance and edit inbound rules to 8080

  - Go to your github click on settings, developer settings, personal access token (tokens classic) then click on generate new token, click on generate new token classic then when giving permissions tick on repo and users then copy the generated code

  - Copy ur public ip and paste on ur browser with semi colon 8080 then copy the stuff on bracket and go to ur vscode terminal and type cat and paste the stuff then hit enter to get the password

  - After setting up your jenkins, go to available plugins and search for blue ocean then install it

    ![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d2d10247-863d-4cd7-9bfc-3c7586064ae8)

  - Go to your dashboard on jenkins and click on blue ocean so that it will open on another tab

    ![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4ff9c2aa-e6ba-4a05-bf8c-e78251a146ec)

 - When it opens click on create a new pipeline click on github then on github token put the token u collected from github then choose the repo u want to work with

   ![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/04d5f901-19ae-40e1-9f31-7f47ebc0d43a)

   ![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f60c679f-9246-4389-87ee-81fe2bed4ef8)

 - At the top in ur vscode click on open folder






































