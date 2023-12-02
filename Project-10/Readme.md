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

create the partition for the remaining two

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

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f1aa827c-b822-4b81-ab08-53b5d3aaf883)

Export the mounts for webservers’ subnet cidr to connect as clients. For simplicity, you will install your all three Web Servers inside the same subnet, but in production set up you would probably want to separate each tier inside its own subnet for higher level of security. To check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

Make sure we set up permission that will allow our Web servers to read, write and execute files on NFS:

sudo chown -R nobody: /mnt/apps

sudo chown -R nobody: /mnt/logs

sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps

sudo chmod -R 777 /mnt/logs

sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/948a0f72-088a-4f62-bc9c-d3a681fc277e)

Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20):

First of all to check your Subnet cidr, go to your NFS server, click on Networking scroll down to subnet click on it then you will see your subnet address

After getting our subnet address type the following command sudo vi /etc/exports when it opens paste the code written below then edit where it says subnet cidr and then put your subnet cidr address

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!  to exit vim

sudo exportfs -arv

Check which port is used by NFS and open it using Security Groups (add new Inbound Rule). type the code below to see the NFS port

sudo rpcinfo -p | grep nfs

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/96ad94ae-25c0-4b7c-b0ee-8539d47433b0)

As you are still on the NFS security group kindly put the following ports

Custom TCP: 2049

CUSTOM TCP: 111

CUSTOM UDP: 2049

CUSTOM UDP: 111

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/68811b3d-08b1-4cf4-b431-f6c6e87fa4e1)

# Step 3: Configure The Database Server

- Install MySQL server

sudo apt install mysql-server

- After doing that, go to your DB server in AWS, go to the security group, for port choose Mysql/Aurora Tcp: 3306 Then for the custom IP, put the NFS subnet cidr IPV4 address

- Next type sudo mysql

- Create a database and name it tooling

create database tooling;

- Create a database user and name it webaccess then attach the NFS subnet cidr address to it

create user 'webaccess'@'172.31.80.0/20' identified by 'password';

- Grant permission to webaccess user on tooling database

GRANT ALL PRIVILEDGES ON tooling.* TO 'webaccess'@'172.31.80.0/20';

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fbbd7280-3dda-4e6b-8ac4-d6348a95abd4)

# Step 4: Prepare The Webservers

- SSH to one of the Webservers launched earlier

- Install NFS client

  sudo yum install nfs-utils nfs4-acl-tools -y

- Mount /var/www/ and target the NFS server’s export for apps

sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d7723ded-ab06-4925-b147-9d6fbee2f12c)

- Verify that NFS was mounted successfully by running df -h. Make sure that the changes will persist on Web Server after reboot:

sudo vi /etc/fstab

add following line;

<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0

- Next Install On The Webserver

sudo yum install mysql -y

- Install Remi’s repository, Apache and PHP

  sudo yum install httpd -y

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo yum install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo yum module  list php -y

sudo yum module reset php -y

sudo yum module enable php:remi-7.4 -y

sudo yum install php php-opcache php-gd php-curl php-mysqlnd -y

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

sudo systemctl restart httpd


# Repeat steps 1-5 for another 2 Web Servers.

- Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly. You can try to create a new file touch test.txt from one server and check if the same file is accessible from other Web Servers.

- Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step 4 under the 'prepare web servers' to make sure the mount point will persist after reboot.

sudo vi /etc/fstab

- Clone the tooling source code from Darey.io Github Account by clicking on code then either copy the HTTPS link or SSH link

- Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

  ![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5f76f5af-51c1-4edf-b87e-458218f39967)

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7d2b7f2f-246a-44b1-a765-c494e9be3162)

- Note 1: Do not forget to open TCP port 80 on the Web Server.

- Note 2: If you encounter 403 Error – check permissions to your /var/www/html folder and also disable SELinux by running sudo setenforce 0

To make this change permanent – open following config file sudo vi /etc/sysconfig/selinux and set SELINUX=disabled then restart httpd.

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/13632cc9-0ee2-4cb0-9b5d-6deec159fab5)

- After doing that, run the below code

sudo systemctl restart httpd

sudo systemctl status httpd

- Update the website’s configuration to connect to the database by running sudo vi /var/www/html/functions.php file 

- Apply tooling-db.sql script to your database using this command mysql -h (Db-private-ip) -u webaccess -p tooling < tooling-db.sql

- Next edit the Mysqld.cnf file

  sudo vi /etc/mysql/mysql.conf.d//mysqld.cnf

sudo systemctl restart mysql

sudo systemctl status mysql

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/452dda71-33f4-4a6c-b9cc-e0c2350e94d8)

- Open the website in your browser http://(Webserver-public-ip)/index.php and make sure you can login into the website






























































