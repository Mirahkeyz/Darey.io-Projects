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









































































