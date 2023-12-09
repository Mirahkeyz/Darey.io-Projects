# AUTOMATING ANSIBLE CONFIGURATION

# Ansible Client as a Jump Server (Bation Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. The webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

# TASKS

- Install and configure Ansible client to act as a Jump Server/Bastion Host

- Create a simple Ansible playbook to automate servers configuration

  # STEP 1: LAUNCH AN INSTANCE AND INSTALL ANSIBLE.

  1. Launch an instance and name it Jenkins-Ansible
 
     ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/675b0eac-67b0-45aa-abb0-01573267dd20)

  3. Then go to your powershell and install Ansible
 
  - sudo apt update
 
  - sudo apt install ansible -y

   ![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/37b0141a-f485-4f5c-a148-b5b6d21acd15)

  - touch ansible.config

    ![SNIPE 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6be61fc6-cae6-4f70-b200-a024cca0d511)

 # STEP 2: INSTALL JENKINS ON THE POWERSHELL 

 1. Go to jenkins.io on your browser then click on ubuntu then follow the commands one after the other and be pasting them on your powershell till you install jenkins

2.  Go to your instance on AWS that is running and edit its inbound rule and place the custom TCP at 8080 from anywhere

# STEP 3: CREATE A REPO ON GITHUB (Ansible-config-mgt)

  - create a new repo on github you can name it ansible-config-mgt

  - click on settings

  - click on webhooks then click on add webhooks then place your instance public address it should look like this http://Public ip:8080/github-webhook/

  - still on the webhooks settings, next click on the drop down and select application/json then click on create webhooks

# STEP 4: CONFIG JENKINS TO BUILD JOB

   - Take your instance public ip and place on your browser e.g http://Publicip:8080 then when it opens copy the stuff in the bracket and go to your powershell and sudo cat it to get the password

   - when it opens click on install suggested plugins

   - when jenkins finally opens and you are getting 403 error anytime you are about do something, just simply click on manage jenkins then click on plugins then click on available plugins then search for strict crumb user and install it.

   - after installing strict crumb user restart your jenkins then go to manage jenkins again and click on security and click on CRSF dropdown and select strict crumb user then click on apply and then click on save. then restart your jenkins

   - click on create a new job and give the name as Ansible and select freestyle project

   -  when it opens for GENERAL settings look at the screenshot below

     ![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bf8b2c85-cdbc-4294-b42b-3267e45931a9)

     

