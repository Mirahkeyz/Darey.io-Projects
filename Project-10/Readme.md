# DEVOPS TOOLING WEBSITE SOLUTION

# Step 1: LAUNCH ALL THE SERVERS NEEDED IN EC2 INSTANCE 

Go to the AWS EC2 instance and launch 3 Web servers using RHEL 8 AMI, 1 NFS server USING RHEL 8 AMI and 1 Database server using UBUNRU 22.0.4 AMI

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/67789d1a-c59b-4665-9ae7-ebd7990c4eef)

After launching the EC2 instances, go to the Elastic Block Store and click on create volumes then create 3 volumes. After creating the volumes, attach it to the NFS EC2 instances.

# Step 2: SSH into the NFS server

After you have ssh to the NFS server on gitbash, type lsblk to see the volumes attached

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fef4c154-7e8b-4ceb-8a06-ad83fdb876b6)

Next use the gdisk utility to create partion for the three volumes

sudo gdisk /dev/xvdf1

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/12122a72-9990-4df5-b240-e9879df7b85e)

create the partion for the remaining two

Next install LVM2 by running sudo yum install lvm2 then carry out the process as in the last project.

Instead of formating the disks as ext4 you will have to format them as xfs

sudo mkfs -t xfs /dev/webdata-vg/lv-apps
sudo mkfs -t xfs /dev/webdata-vg/lv-logs
sudo mkfs -t xfs /dev/webdata-vg/lv-opt

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c0e85864-9f9d-4aed-957e-bd46226625e8)

Create mount points on /mnt directory for the logical volumes as follow:

sudo mkdir /mnt/apps

sudo mkdir /mnt/logs

sudo mkdir /mnt/opt

sudo mount /dev/webdata-vg/lv-apps /mnt/apps

sudo mount /dev/webdata-vg/lv-logs /mnt/logs

sudo mount /dev/webdata-vg/lv-opt /mnt/opt

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b978e9f5-08a4-44dd-aa11-37c7c384d76b)

Install NFS server, configure it to start on reboot and make sure it is u and running

sudo yum -y update

sudo yum install nfs-utils -y

sudo systemctl start nfs-server.service

sudo systemctl enable nfs-server.service

sudo systemctl status nfs-server.service

Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security. To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

sudo chown -R nobody: /mnt/apps

sudo chown -R nobody: /mnt/logs

sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps

sudo chmod -R 777 /mnt/logs

sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service

Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20):

First of all to check your Subnet cidr, go to your NFS server, click on Networking scroll down to subnet click on it then you will see your subnet address

After getting our subnet address type the following command sudo vi /etc/exports when it opens paste the code written below then edit where it says subnet cidr and then put your subnet cidr address










































