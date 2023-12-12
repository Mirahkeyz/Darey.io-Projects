# Ansible Refactoring, Assignments & Imports

# Step 1 – Jenkins job enhancement

let us make some changes to our Jenkins job – now every new change in the codes creates a separate directory which is not very convenient when we want to run some commands from one place. Besides, it consumes space on Jenkins server with each subsequent change. Let us enhance it by introducing a new Jenkins project/job – we will install Copy Artifact plugin.

1. Go to Powershell and SSH then create a new directory called ansible-config-artifact – we will store all artifacts after each build with the command below

   sudo mkdir /home/ubuntu/ansible-config-artifact

2. Change permissions to this directory, so Jenkins could save files there. use the command below

  sudo chown -R jenkins:jenkins /home/ubuntu/ansible-config-artifact

3. Go to Jenkins web console -> Manage Jenkins -> Manage Plugins -> on Available tab search for Copy Artifact and install this plugin without restarting Jenkins

   ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c719db6c-95ad-412e-9f65-59161b81aacc)

4. Create a new Freestyle project and name it save_artifacts. The main idea of save_artifacts project is to save artifacts into /home/ubuntu/ansible-config-artifact directory.

 ![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9281134c-6f70-41ad-acee-0fa6770de22a)

5. follow the pictures below to set up the new job

   ![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/54ffd596-8cc7-4aef-ae81-efa252897b2d)

   ![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/71e4e82b-6dcc-438f-8a53-e21c7bc39b6f)

6. Test your set up by making some change in README.MD file inside your ansible-config-mgt repository (right inside main branch).


# Step 2 – Refactor Ansible code by importing other playbooks into site.yml

1. Go to your vscode terminal switch to the development branch

2. Create a folder called static-assignments then on your vscode drag the common.yml file from playbooks folder into static-assignments folder then right click on static-assignments folder and create a new file called common-del.yml. The static-assignments folder is where all other children playbooks will be stored. This is merely for easy organization of your work. It is not an Ansible specific concept, therefore you can choose how you want to organize your work. You will see why the folder name has a prefix of static very soon

3. Right click on playbooks and create a file called site.yml. inside the site.yml file import the common.yml playbook on it

   ---
- hosts: all
- import_playbook: ../static-assignments/common.yml

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0a6161f9-9f12-4120-8fd1-3918f82380a1)

4. Update git with the latest changes

 git status
 
git add .

git commit -m ("with the message")

git push -u origin development

5. Go to your powershell and type below
   
Eval ‘ssh-agent -s’

ssh-add (put ur keypair)

ssh-add -l

Copy your jenkins-ansible public ip and type the below
    
ssh -A ubuntu@public-ip 

cd into ansible-config-artifact or ansible-config-mgt

 run the playbook ansible-playbook -i inventory/dev playbooks/site.yml

5. Since i need to apply some tasks to your dev servers and wireshark is already installed, go to common-del.yml. In this playbook, configure deletion of wireshark utility.

---
- name: update web and nfs servers
  hosts: webservers and nfs
  remote_user: ec2-user
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    yum:
      name: wireshark
      state: removed

- name: update LB and DB servers
  hosts: lb, db 
  remote_user: ubuntu
  become: yes
  become_user: root
  tasks:
  - name: delete wireshark
    apt:
      name: wireshark-qt
      state: absent
      autoremove: yes
      purge: yes
      autoclean: yes


6. ---
- hosts: all
- import_playbook: ../static-assignments/common.yml


7. update git with the latest code

8. Go to your powershell and Run the ansible playbook

ansible-playbook -i inventory/dev.yml playbooks/site.yml

# Step 3 – Configure UAT Webservers with a role ‘Webserver’

1. Launch 2 fresh EC2 instances using RHEL 8 image, we will use them as our uat servers, so give them names accordingly – Web1-UAT and Web2-UAT.

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/69accc07-63b5-4924-aa72-30841a37dc7c)

2.  Go to your vscode terminal and create a directory and name it roles then cd into the roles directory

3. Then inside the roles directory put the code below
   
ansible-galaxy init webserver

Note: wsl must be installed already to run the above command to check if wsl is installed type wsl --version

4. Update your inventory ansible-config-mgt/inventory/uat file with IP addresses of your 2 UAT Web servers

[uat-webservers]
<Web1-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

<Web2-UAT-Server-Private-IP-Address> ansible_ssh_user='ec2-user' 

5. Update git with latest commit

6.  Go to your powershell and type sudo vi /etc/ansible//ansible.cfg then scroll down to roles then uncomment it and paste the roles path copied from vscode meaning you need to have already right clicked on roles on vscode and click on copy path

7.  Post the below code on the main.yml file under task in the role directory and for the clone path in the code below, paste the link to the tooling website you already forked.

   ---
- name: Install apache
  become: true
  ansible.builtin.yum:
    name: "httpd"
    state: present

- name: Install git
  become: true
  ansible.builtin.yum:
    name: "git"
    state: present

- name: clone a repo
  become: true
  ansible.builtin.git:
    repo: https://github.com/Mirahkeyz/tooling.git
    dest: /var/www/html
    force: yes

- name: copy html content to one level up
  become: true
  command: cp -r /var/www/html /var/www/

- name: Start service httpd, if not started
  become: true
  ansible.builtin.service:
    name: httpd
    state: started

- name: recursively remove /var/www/html directory
  become: true
  ansible.builtin.file:
  path: /var/www/html
  state: absent


# Step 4 – Reference ‘Webserver’ role

1. Right click on the static-assignment folder and create a file called uat-webservers.yml. This is where you will reference the role. paste the code below

   ---
- hosts: uat-webservers
  roles:
     - webserver
 
  2.  Paste the below code inside the site.yml file and comment anyother one there

   ---
- hosts: uat-webservers
- import_playbook: ../static-assignments/uat-webservers.yml

3.  update git with latest changes

4. Go to powershell and cd into your ansible-config.mgt file and run the playbook
   
ansible-playbook -i /inventory/uat playbooks/site.yml

30. Copy any of the uat public ip from aws. e.g public-ip/index.php

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/85e944e7-8839-4c9c-bc5a-4b17c81ffbba)






























    





















