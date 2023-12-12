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














