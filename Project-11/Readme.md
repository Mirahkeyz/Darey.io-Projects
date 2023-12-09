# AUTOMATING ANSIBLE CONFIGURATION

# Ansible Client as a Jump Server (Bation Host)

A Jump Server (sometimes also referred as Bastion Host) is an intermediary server through which access to internal network can be provided. The webservers would be inside a secured network which cannot be reached directly from the Internet. That means, even DevOps engineers cannot SSH into the Web servers directly and can only access it through a Jump Server – it provide better security and reduces attack surface.

# TASKS

- Install and configure Ansible client to act as a Jump Server/Bastion Host

- Create a simple Ansible playbook to automate servers configuration

  # STEP 1: LAUNCH AN INSTANCE AND INSTALL ANSIBLE.

  1. Launch an ubuntu instance and name it Jenkins-Ansible
 
     ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/675b0eac-67b0-45aa-abb0-01573267dd20)

  2. Launch other servers which are NFS server(Rhel 9 ami), WEB 1 and WEB 2 servers(Rhel 9 ami for each), Db server(Rhel 9 ami), LB server(ubuntu 22.04 ami) and they should use the same key pair with the jump server we created before

     ![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b6324808-74cb-47d1-875d-a6c7139cb556)


  3. Then go to your powershell and install Ansible using the keypair of the jump server 
 
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

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d5a5c95f-646d-4c43-93e7-bed17b92b3e5)

  - for source code management select Git then for repository url place the repo link you are working on in the box

  - specify */main branch

    ![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5bc05fe6-7737-4c94-81b8-3697ea414668)

  - For Build Triggers seting select Github hook trigger fot Gitscm polling

    ![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8d184b4d-cbaf-4493-969e-feb84b5f5208)

  - For Post Build Action select Archive the Arctifacts then type ** on the box

    ![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9f645d42-c144-45f0-bbae-3b3a11138e09)

   # Note: everytime I stop/start my my Ansible server, I will have to reconfig github webhook Ip address. To avoid this, I can allocate an Elastic Ip to the server.Elastic IP is only free when allocated to an EC2 instance, so remenber to release Elastic Ip once I terminate Instance.

   # STEP 4: PREPARING THE DEVELOPMENT AREA USING VSCODE

  - Create a new folder on Gitbash and name it ansible-config-mgt then cd into that folder and type code . to open vscode

  - Go to your vscode terminal and switch to development branch and create two separate directories

    git checkout -b development

    mkdir playbooks

    mkdir inventory

    - Right click on playbooks and create new file and name it common.yml
   
    - Right click on inventory and create different files one after the other and name it dev, staging, uat, prod
   
    - go to the vscode terminal and type the below codes
   
    git add .

    git commit -m "initial commit"

    git remote add origin https://github.com/mirahkeyz/ansible-config-mgt/tree/development     pls add your own repo link and github name here link here

   - Go to github and merge the pull request to main branch

 # STEP 5: Setup SSH agent using windows power shell terminal using the following script

   - open the link https://learn.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse?tabs=powershell and follow the procedures or follow mine below

   - First we are going to install OPENSSH or check if it is already installed with the code below

     Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

     - Then, install the server or client components as needed with the code below
    
    # Install the OpenSSH Client
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

    # Install the OpenSSH Server
Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

   - To start and configure OpenSSH Server for initial use, open an elevated PowerShell prompt (right click, Run as an administrator), then run the following commands to start the sshd service:

         # Start the sshd service
Start-Service sshd

    # OPTIONAL but recommended:
Set-Service -Name sshd -StartupType 'Automatic'

    # Confirm the Firewall rule is configured. It should be created automatically by setup. Run the following to verify
if (!(Get-NetFirewallRule -Name "OpenSSH-Server-In-TCP" -ErrorAction SilentlyContinue | Select-Object Name, Enabled)) {
    Write-Output "Firewall Rule 'OpenSSH-Server-In-TCP' does not exist, creating it..."
    New-NetFirewallRule -Name 'OpenSSH-Server-In-TCP' -DisplayName 'OpenSSH Server (sshd)' -Enabled True -Direction Inbound -Protocol TCP -Action Allow -LocalPort 22
} else {
    Write-Output "Firewall rule 'OpenSSH-Server-In-TCP' has been created and exists."
}

    - Next we are To generate key files using the Ed25519 algorithm, run the following command

    ssh-keygen -t ed25519

      You can press Enter to accept the default, or specify a path and/or filename where you would like your keys to be generated. At this point, you'll be prompted to use a passphrase to encrypt your private key files. The passphrase can be empty but it's not recommended. The passphrase works with the key file to provide two-factor authentication. For this example, we're leaving the passphrase empty.

      
  # STEP 6: UPDATE THE INVENTORY/Common.yml file created and Playbook/dev file created

     use the code below to update the dev file under playbook. copy the private ips from the servers we created before and replace it on the code

     [nfs]
<NFS-Server-Private-IP-Address> ansible_ssh_user=ec2-user

[webservers]
<Web-Server1-Private-IP-Address> ansible_ssh_user=ec2-user

<Web-Server2-Private-IP-Address> ansible_ssh_user=ec2-user

[db]
<Database-Private-IP-Address> ansible_ssh_user=ec2-user 

[lb]
<Load-Balancer-Private-IP-Address> ansible_ssh_user=ubuntu

- use the picture below to update the common.yml file under inventory

  ![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/460af0af-187d-4d9f-8704-3a05c2cac56f)

# STEP 7: Update git with the latest code

  

    


     

