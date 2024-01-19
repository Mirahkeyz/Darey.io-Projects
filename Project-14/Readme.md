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

  ![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7299186c-5015-4998-8bd1-48110c95b968)

  - Spin up a vscode terminal
 
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

 - Click on vscode terminal and install git by typing sudo yum install git -y

 - Copy the repo link (ansible-config-mgt) and go back to vscode terminal and type git clone and paste the link

 - When it clones finish you will see it at the top so right click on it and create a folder called deploy. Then under deploy i will create a file called jenkinsfile then paste the below script inside the jenkins file
    
   
           pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }
  }
}

- Go back to jenkins dashboard, click on ansible-config-mgt then click on configuration

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/abd6293c-2812-4661-922c-1405c1fe5e93)

- Under github credentials add ur username and password

- Click on build configuration click on script path and specify were your jenkins file is on vscode e.g deploy/jenkinsfile then click on apply and save

 ![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/494c855a-0107-408d-a13e-feaf31b62145)

- Go to vscode terminal and type git config --global user.name Mirahkeyz

- Type again git config --global user.email Mirahkeys@gmail.com

- cd into ansible-config-mgt

       git branch

       git add .

       git  commit -m added jenkinsfile
    
       git push

- Go bk to ansible dashboard on jenkins and click on scan repo now and main will appear and it will build successfully

  ![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0dcb2c02-adf7-4ea6-8297-8e5df87df5e3)

  ![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/af74c908-8499-44c5-87a3-9d23e6337300)

  ![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d6562c40-9fe9-486d-a9df-df1022bc5c60)

- Create a new git branch on vscode terminal and name it feature/jenkinspipeline-stages

- then go to your jenkinsfile and replace it with the script below

  pipeline {
    agent any

  stages {
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }
  }
}

- Then update git with latest changes

- Go to jenkins and click on scan repo now to display the new branch and the build process

  ![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c6ff7f5e-b807-4927-86f0-fbc047fd774b)


- Go to github Create a pull request to merge the latest code into the main branch

-  Go to vscode terminal and type git switch main follow by git pull

- Go to jenkinsfile on vscode and replace it with below

  pipeline {
    agent any

  stages {
    stage("Initial cleanup"){
      steps {
        dir("${WORKSPACE}") {
          deleteDir()
        }
      }
    }
    
    stage('Build') {
      steps {
        script {
          sh 'echo "Building Stage"'
        }
      }
    }

    stage('Test') {
      steps {
        script {
          sh 'echo "Testing Stage"'
        }
      }
    }

    stage('Package') {
      steps {
        script {
          sh 'echo "Packaging Stage"'
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          sh 'echo "Deploying Stage"'
        }
      }
    }

    stage("Clean up workspace after build") {
          steps {
            cleanWs()
            }
    }
  }
}

-  Update git with latest changes

-  Now that you have a broad overview of a typical Jenkins pipeline. Let us get the actual Ansible deployment to work by:
Installing Ansible on Jenkins. Install the following dependencies if not yet installed:
sudo yum install -y ansible-2.9.25
python3 -m pip install --upgrade setuptools
python3 -m pip install --upgrade pip
python3 -m pip install PyMySQL
python3 -m pip install mysql-connector-python
python3 -m pip install psycopg2-binary
ansible-galaxy collection install community.postgresql
ansible-galaxy collection install community.mysql

- Go to jenkins, available plugins, search for ansible and install it

  ![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6f41ac55-2e54-434a-8574-3d50cb6d8089)

  
- Go to credentials on jenkins, click on system global, click on Add credentials, under kind select SSH username with private key, under id write key-pair, under username write ec2-user, under private key choose enter directly then a box will open go to the location were ur key pair is using git bash and type cat and the name of the key pair then copy everything displayed go back to jenkins and paste it on the box then click on create
  
- Go to your vscode terminal and type which ansible to know the path of ansible e.g /usr/bin/ansible then go to jenkins, tools, scroll down to where you see add ansible then click on it then give the name as ansible and path as /usr/bin/
  
- go to your jenkinsfile and replace everything with the script below and make changes with your own details



pipeline {
  agent any


  environment {
      ANSIBLE_CONFIG="${WORKSPACE}/deploy/ansible.cfg"
    }


  parameters {
      string(name: 'inventory', defaultValue: 'dev',  description: 'This is the inventory file for the environment to deploy configuration')
    }


  stages{
      stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }


      stage('Checkout SCM') {
         steps{
            git branch: 'main', url: 'https://github.com/Mirahkeyz/ansible-config.git'
         }
       }


      stage('Prepare Ansible For Execution') {
        steps {
          sh 'echo ${WORKSPACE}' 
          sh 'sed -i "3 a roles_path=${WORKSPACE}/roles" ${WORKSPACE}/deploy/ansible.cfg'  
        }
     }


      stage('Run Ansible playbook') {
        steps {
           ansiblePlaybook become: true, colorized: true, credentialsId: 'private-key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory/${inventory}', playbook: 'playbooks/site.yml'
         }
      }


      stage('Clean Workspace after build'){
        steps{
          cleanWs(cleanWhenAborted: true, cleanWhenFailure: true, cleanWhenNotBuilt: true, cleanWhenUnstable: true, deleteDirs: true)
        }
      }
   }


}

-  Go to your jenkins ansible project and click on pipeline syntax then under sample steps select ansible Playbook: invoke an ansible playbook, under playbook file path in workspace type playbooks/site.yml under inventory file path in workspace type inventory/dev, under SSH connection credentials select ec2-user, tick on use become, tick on disable the SSH key check, tick on colorized output then click on generate pipeline script then copy the script then go to the jenkinsfile on vscode under run ansible playbook and paste urs there

- Go to ur vscode project right click on deploy and create a file called ansible.cfg then paste the below script

 [defaults]
timeout = 160
callback_whitelist = profile_tasks
log_path=~/ansible.log
host_key_checking = False
gathering = smart
ansible_python_interpreter=/usr/bin/python3
allow_world_readable_tmpfiles=true


[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=30m -o ControlPath=/tmp/ansible-ssh-%h-%p-%r -o ServerAliveInterval=60 -o ServerAliveCountMax=60 -o ForwardAgent=yes
 
- Click on sites.yml under playbook then uncomment the nginx and comment the rest then for uat-webservers.yml then comment everything and uncomment nginx then for dev under inventory uncomment nginx and comment the rest then spin up an instance and name it nginx then copy the private ipv4 and replace it
  
- Update git with the latest changes

# CI/CD PIPELINE FOR TODO APPLICATION

# Phase 1 – Prepare Jenkins

============================

1. Fork the repository below into your GitHub account
https://github.com/darey-devops/php-todo.git

2. On you Jenkins server, install PHP, its dependencies and Composer tool (Feel free to do this manually at first, then update your Ansible accordingly later)

yum module reset php -y

yum module enable php:remi-7.4 -y

yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json

systemctl start php-fpm

systemctl enable php-fpm

sudo apt install -y zip libapache2-mod-php phploc php-{xml,bcmath,bz2,intl,gd,mbstring,mysql,zip}

3. Install Jenkins plugins

- Plot plugin

- Artifactory plugin

4. Spin up another server that will host the jfrog artifactory

![Snipe 68](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3cea70dc-a594-48d1-a221-99e99e670a5c)

![Snipe 61](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9fbf413f-cd72-47ce-905f-35d3886751fc)

![Snipe 62](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ea148a62-d2ce-46a0-9452-3fd89d1961cb)

![Snipe 63](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/77c7e31d-72dc-41d4-8432-6d351fa96f90)

![Snipe 64](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a1cfaf8f-8965-47a2-b184-11c60ef397ac)

![Snipe 65](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3d6652da-4153-4bef-8556-3c5b547eeff0)

We will use plot plugin to display tests reports, and code coverage information. The Artifactory plugin will be used to easily upload code artifacts into an Artifactory server.

5. In Jenkins UI configure Artifactory

   ![Snipe 66](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/473e6052-19c8-4650-9284-32456dde9164)

  ![Snipe 67](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b763644d-be72-452a-a6e2-ffb7adcd5c2d)

# Phase 2 – Integrate Artifactory repository with Jenkins

1. Create a dummy Jenkinsfile in the repository

2. Using Blue Ocean, create a multibranch Jenkins pipeline

3. On the database server, create database and user

4. Install mysql client: sudo apt install mysql -y

5. Login into the DB-server(mysql server) and set the the bind address to 0.0.0.0: sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

6. Create database and user. NOTE: The task of setting the database is done by the MySQL ansible role

Create database homestead;
CREATE USER 'homestead'@'%' IDENTIFIED BY 'sePret^i';
GRANT ALL PRIVILEGES ON * . * TO 'homestead'@'%';

![Snipe 69](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ef08952a-ab30-46ff-9943-5a7828a3ca53)

![Snipe 70](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c911ca2d-cd75-454d-8f80-10062a6a19a4)


1. Update the database connectivity requirements in the file .env.sample

   ![Snipe 71](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c65f6b30-01f0-45f4-86aa-4f1623f141ae)

2. Update Jenkinsfile with proper pipeline configuration
  
  pipeline {
    agent any

  stages {

     stage("Initial cleanup") {
          steps {
            dir("${WORKSPACE}") {
              deleteDir()
            }
          }
        }

    stage('Checkout SCM') {
      steps {
            git branch: 'main', url: 'https://github.com/darey-devops/php-todo.git'
      }
    }

    stage('Prepare Dependencies') {
      steps {
             sh 'mv .env.sample .env'
             sh 'composer install'
             sh 'php artisan migrate'
             sh 'php artisan db:seed'
             sh 'php artisan key:generate'
      }
    }
  }
}

3. We need to install mysql client on the Jenkins server and configure it.

![Snipe 72](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f169d51b-8184-46ae-a5b4-51f1c418e5ac)

![Snipe 73](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ff68c053-c393-4ac5-a9cc-c7c6d2b06679)

![Snipe 74](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b2a8585f-b4a1-4af5-ba11-45c800397a30)

4. Bundle the application code into an artifact (archived package) and upload to Artifactory

     Install Zip: Sudo apt install zip -y

stage ('Package Artifact') {
    steps {
            sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
     }
    }

5. Publish the resulted artifact into Artifactory making sure ti specify the target as the name of the artifactory repository you created earlier

   stage ('Upload Artifact to Artifactory') {
          steps {
            script { 
                 def server = Artifactory.server 'artifactory-server'                 
                 def uploadSpec = """{
                    "files": [
                      {
                       "pattern": "php-todo.zip",
                       "target": "PBL/php-todo",
                       "props": "type=zip;status=ready"

                       }
                    ]
                 }""" 

                 server.upload spec: uploadSpec
               }
            }

        }

![Snipe 75](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c0999638-7e31-46b0-b9c3-7b93b1c104f8)

Deploy the application to the dev environment by launching Ansible pipeline. Ensure you update your inventory/dev with the Private IP of your TODO-server and your site.yml file is updated with todo play.

stage ('Deploy to Dev Environment') {
    steps {
    build job: 'ansible-project/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
    }
  }


 ![Snipe 78](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7f3ddfa9-6b33-4de6-bf04-95b18df970c7)
 

# SONARQUBE INSTALLATION

SONARQUBE INSTALLATION SonarQube is a tool that can be used to create quality gates for software projects, and the ultimate goal is to be able to ship only quality software code.

Despite that DevOps CI/CD pipeline helps with fast software delivery, it is of the same importance to ensure the quality of such delivery. Hence, we will need SonarQube to set up Quality gates. In this project we will use predefined Quality Gates (also known as The Sonar Way). Software testers and developers would normally work with project leads and architects to create custom quality gates.

Setting Up SonarQube On the Ansible config management pipeline, execute the ansible playbook script to install sonarqube via a preconfigured sonarqube ansible role.

![Snipe 76](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/df522c21-7efb-4316-8408-6991ae7391e6)

When the pipeline is complete, access sonarqube from the browser using the <sonarqube_server_url>:9000/sonar

![Snipe 77](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/92aa28b6-7486-4aff-8864-134d08f2dfd6)

# CONFIGURE SONARQUBE AND JENKINS FOR QUALITY GATE

![Snipe 79](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/01cf987c-0408-4c70-b067-77ffafdbca7c)

- Install SonarQube Scanner plugin

  ![Snipe 80](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b46eb98f-b974-417d-b623-eaa2b690d11b)

- Navigate to configure system in Jenkins. Add SonarQube server: Manage Jenkins > Configure System

- To generate authentication token in SonarQube to to: User > My Account > Security > Generate Tokens

- Setup SonarQube scanner from Jenkins – Global Tool Configuration. Go to: Manage Jenkins > Global Tool Configuration

- Update Jenkins Pipeline to include SonarQube scanning and Quality Gate. Making sure to place it before the "package artifact stage" Below is the snippet for a Quality Gate stage in Jenkinsfile.

stage('SonarQube Quality Gate') {
    environment {
        scannerHome = tool 'SonarQubeScanner'
    }
    steps {
        withSonarQubeEnv('sonarqube') {
            sh "${scannerHome}/bin/sonar-scanner"
        }

    }
}

# NOTE: The above step will fail because we have not updated sonar-scanner.properties.

![Snipe 81](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c04da753-aa39-4290-8fd4-9aba873550a5)

- Configure sonar-scanner.properties – From the step above, Jenkins will install the scanner tool on the Linux server. You will need to go into the tools directory on the server to configure the properties file in which SonarQube will require to function during pipeline execution. cd /var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQubeScanner/conf/.

- Open sonar-scanner.properties file: sudo vi sonar-scanner.properties

- Add configuration related to php-todo project

sonar.host.url=http://<SonarQube-Server-IP-address>:9000
sonar.projectKey=php-todo
#----- Default source code encoding
sonar.sourceEncoding=UTF-8
sonar.php.exclusions=**/vendor/**
sonar.php.coverage.reportPaths=build/logs/clover.xml
sonar.php.tests.reportPath=build/logs/junit.xml 

![Snipe 82](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a6f9660a-e382-4dc3-9fe1-5ff87d662e2f)































































   













    


                    


































































