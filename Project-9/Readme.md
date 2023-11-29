# Implementing Wordpress Project With LVM

# STEP 1: Launch An EC2 Instance That Will Serve As A Web Server

we are going to create an EC2 instance that will serve as web server.

We set this up by doing the following:

- create an account on AWS.

- we create an EC2 instance by selecting “REDHAT ENTERPRISE LINUX 9” from Amazon Machine Image(AMI)(free tier). We will select a Redhat AMI since we are using a centos and not ubuntu.

- we select “t2.micro(free tier eligible)”.

- Select our keypair.

- then go to the security group and select “a security group” review and launch.

  This launches us into the instance as shown in the screenshot:

  ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/70c68eac-31eb-4420-a216-14ddb477460e)

# Step 2: Create Volume And Attach It To The Instance

 we go to EC2 and click on "volume" under Elastic block store(EBS).

Click on "create volume"

then change size to 10GiB and click on create volume afterwards.

We do the same process to create three EBS volumes.

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/649e71b9-fcf9-468a-b6e6-9e93dab792c8)

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/481b60c3-f62e-4de2-8c7e-7373f4782237)

we then attach the EBS volumes we created to the webserver Ec2 instance.

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ad5468a3-5239-40a3-a202-e4838495992e)

then edit the "instance info" with the "instance id" and "device name info" to "/dev/xvdf", "/dev/xvdg" and /dev/xvdh" for the three volumes.

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/68057f41-2279-4bd8-a41c-d8a7fa9f7863)

then click on attach. do this for the three Ebs volumes created.

# Step 3: Connecting To The Instance Using The Terminal

- ssh inside the directory you downloaded your key pair

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2a328cd2-1fcb-4057-9a7d-4af243f1a744)

- Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.

  ![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3aca0026-e0b2-458a-b0fc-bd5926bcaaab)

  Use df -h command to see all mounts and free space on your server.

Use gdisk utility to create a single partition on each of the 3 disks

$ sudo gdisk /dev/xvdf

type "?" to display the available options. Then "p" "which represents print the partition table".

From the options displayed, "n" represents "add a new partition".

Type "n" then "p" to display the new partiton table Type "w" to write table to disk.

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6c1ff993-44a6-443b-b1cc-5664b5ddca79)

Repeat the process for the the three disks i.e /dev/xvdf, /dev/xvdg. /dev/xvdh.

Use lsblk utility to view the newly configured partition on each of the 3 disks.

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/961a64e2-bd5d-4d05-b8cc-e051a4b9708f)

# Step 4: Install LVM

Install lvm2 package using

$ sudo yum install lvm2

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c506dfee-4dcd-4e01-8051-d4b1b08e0063)

Run

$ sudo lvmdiskscan to check for available partitions.

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ed3628b7-9513-488a-bae0-8bf4f6fe46eb)

Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.

$ sudo pvcreate /dev/xvdf1

$ sudo pvcreate /dev/xvdg1

$ sudo pvcreate /dev/xvdh1

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/942cf679-37bf-4a76-ba5c-f4566d922182)

Verify that your Physical volume has been created successfully by running

$ sudo pvs

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9e45e132-ab09-457e-bbb3-91f8fd983d65)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

$ sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f77f4525-957d-4c0c-b721-8902eb23a0bb)

Verify that your VG has been created successfully by running

$ sudo vgs

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv (Use the remaining space of the PV size).

The apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

$ sudo lvcreate -n apps-lv -L 14G webdata-vg

$ sudo lvcreate -n logs-lv -L 14G webdata-vg

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/96f98d2c-b127-4c51-b4db-f07e16df24a4)

Verify that your Logical Volume has been created successfully by running

$ sudo lvs

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9de04dd3-e53e-4f01-9125-3c374ca2f26b)

Verify the entire setup

$ sudo vgdisplay -v #view complete setup - VG, PV, and LV

$ sudo lsblk

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1b93efb5-7b5c-4fd0-8990-13421d2497cc)

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7ed838c2-62c0-4e1f-92ff-c1d81d16fdc7)

Use "mkfs.ext4" to format the logical volumes with "ext4" filesystem

$ sudo mkfs -t ext4 /dev/webdata-vg/apps-lv

$ sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/717dcb79-ec80-44e8-8092-93ae2e35fde3)

Create "/var/www/html" directory to store website files

$ sudo mkdir -p /var/www/html

Create /home/recovery/logs to store backup of log data

$ sudo mkdir -p /home/recovery/logs

Mount "/var/www/html" on "apps-lv" logical volume

$ sudo mount /dev/webdata-vg/apps-lv /var/www/html/

Use rsync utility to backup all the files in the log directory "/var/log" into "/home/recovery/logs" (This is required before mounting the file system).

$ sudo rsync -av /var/log/. /home/recovery/logs/

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/811022ef-f26b-4d0b-acc8-86fcae3378ac)

Mount "/var/log" on "logs-lv" logical volume. (all the existing data on "/var/log" will be deleted.

$ sudo mount /dev/webdata-vg/logs-lv /var/log

Restore log files back into "/var/log" directory.

$ sudo rsync -av /home/recovery/logs/. /var/log

![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c6fb3bc1-c95c-444d-a168-66bf08caf461)

Update "/etc/fstab file" so that the mount configuration will persist after restart of the server.

UPDATE THE "/ETC/FSTAB" FILE.

The UUID of the device will be used to update the /etc/fstab file;

$ sudo blkid

![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1634614e-edb7-4b11-9109-36973007df99)

Open the "/etc/fstab"

$ sudo vi /etc/fstab

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.

Test the configuration by running this command. There will be no errors if everything is ok.

$ sudo mount -a

Reload the daemon

$ sudo systemctl daemon-reload

Verify your setup by running

df -h

![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/03423f91-8458-4279-9220-d34281e386a1)

# Step 5: Prepare The Database Server

Launch a second RedHat EC2 instance that will have a role – ‘DB Server’ Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

The apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

$ sudo lvcreate -n db-lv -L 14G webdata-vg

$ sudo lvcreate -n logs-lv -L 14G webdata-vg

Verify that your Logical Volume has been created successfully by running

$ sudo lvs

![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f28ee0ec-e93e-430a-b119-55bd4b845aaf)

Verify the entire setup

$ sudo vgdisplay -v #view complete setup - VG, PV, and LV

![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/51adbaed-6df1-4daf-afa4-c27bb9a8b437)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem.

$ sudo mkfs -t ext4 /dev/webdata-vg/db-lv

$ sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

Create db directory to store database files

$ sudo mkdir db

Create /home/recovery/logs to store backup of log data

$ sudo mkdir -p /home/recovery/logs

Mount db/ on db-lv logical volume

$ sudo mount /dev/webdata-vg/db-lv db/

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system).

$ sudo rsync -av /var/log/. /home/recovery/logs/

Mount /var/log on logs-lv logical volume. (all the existing data on /var/log will be deleted.)

$ sudo mount /dev/webdata-vg/logs-lv /var/log

Restore log files back into /var/log directory

$ sudo rsync -av /home/recovery/logs/. /var/log

The UUID of the device will be used to update the /etc/fstab file;

$ sudo blkid

![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4eae644f-344e-4d52-9e1e-00b9729e1082)

Open the "/etc/fstab" file.

$ sudo vi /etc/fstab

Update "/etc/fstab" in this format using your own UUID and rememeber to remove the leading and ending quotes.

![Snipe 28](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a980e310-8020-493e-ba32-a3c85f5a9e63)

Test the configuration for errors.

$ sudo mount -a

Reload daemon

$ sudo systemctl daemon-reload

Verify your setup by running

$ df -h

![Snipe 29](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1df1db18-61d0-47b5-abfe-24b7f2bc092a)

# Step 6: Install Wordpress On Your Webserver EC2 Instance

Update the repository

$ sudo yum -y update

Install wget, Apache and it’s dependencies

$ sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

![Snipe 30](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f52f4b59-e6ec-4c82-80fc-68069127b219)

Start Apache

$ sudo systemctl start httpd

Enable Apache

$ sudo systemctl enable httpd

Verify Apache status

$ sudo systemctl status httpd

![Snipe 31](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/70ebfe1b-9999-4def-b259-187bd40c4f42)

To install PHP and it’s depemdencies

$ sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ sudo yum module list php

$ sudo yum module reset php

$ sudo yum module enable php:remi-7.4

$ sudo yum install php php-opcache php-gd php-curl php-mysqlnd

Start php and set policies

$ sudo systemctl start php-fpm

Enable php

$ sudo systemctl enable php-fpm

verify php status

$ sudo systemctl status php-fpm

$ sudo setsebool -P httpd_execmem 1

Restart Apache

$ sudo systemctl restart httpd

![Snipe 32](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ff42cfee-8e4f-4645-8a4c-3b3c7abc5a93)

Copy the IP address of the webserver to the browser to see that the apache is working properly

![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9e2ab851-0854-42ed-a172-a63cab0376c7)

Download wordpress and copy wordpress to "var/www/html"

Create directory wordpress and cd into the directory.

$ mkdir wordpress

$ cd   wordpress

Download the wordpress file

$ sudo wget http://wordpress.org/latest.tar.gz

Unzip the file

$ sudo tar xzvf latest.tar.gz

![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f6a257dc-256d-45d8-9cfc-977fc23a51b8)

$sudo rm -rf latest.tar.gz

Copy "wordpress/wp-config-sample.php" into "wordpress/wp-config.php"

N/B: wordpress/wp-config.php" will be created.

$ sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php

Copy wordpress into "/var/www/html"

$ sudo cp -R wordpress/* /var/www/html/

Configure SELinux Policies

$ sudo chown -R apache:apache /var/www/html/

$ sudo chcon -t httpd_sys_rw_content_t /var/www/html/ -R

$ sudo setsebool -P httpd_can_network_connect=1

$ sudo setsebool -P httpd_can_network_connect_db 1

# Step 7: Install Mysql On Your DB EC2 Instance

Update the repository

$ sudo yum update -y

Install mysql-server

$ sudo yum install mysql-server

![Snipe 35](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bbfea08f-3f3d-4cb5-9607-23db00c19918)

Verify that the service is up and running, if it is not running, restart the service and enable it so it will be running even after reboot:

$ sudo systemctl restart mysqld

$ sudo systemctl enable mysqld

$ sudo systemctl status mysqld

![Snipe 36](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/711ca678-8e50-46dc-99bb-2a5015102fd7)

Configure DB to work with WordPress

$ sudo mysql

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

mysql> exit;

$ sudo mysql_secure_installation

![Snipe 37](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2b626766-9532-4d06-b042-ad912abdc6d8)







































































































































































































