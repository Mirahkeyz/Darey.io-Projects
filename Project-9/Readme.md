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











































