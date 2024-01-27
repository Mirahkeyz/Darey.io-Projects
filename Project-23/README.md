# AWS Cloud Solution For 2 Company's Websites Using A Reverse Proxy Technology

WARNING: This infrastructure set up is NOT covered by AWS free tier. Therefore, ensure to DELETE ALL the resources created immediately after finishing the project. Monthly cost may be shockingly high if resources are not deleted. Also, it is strongly recommended to set up a budget and configure notifications when your spendings reach a predefined limit. Watch this video to learn how to configure AWS budget.

In this project, we will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website tooling for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

# Starting Off AWS Project

1. Properly configure your AWS account and Organization Unit Watch How To Do This here
   
- Create an AWS Master account. (Also known as Root Account)

- Within the Root account, create a sub-account and name it DevOps. (You will need another email address to complete this)

- Within the Root account, create an AWS Organization Unit (OU). Name it Dev. (We will launch Dev resources in there) Move the DevOps account into the Dev OU.

- Login to the newly created AWS account using the new email address.

- Create a free domain name for your fictitious company at Freenom domain registrar here.

- Create a hosted zone in AWS, and map it to your free domain from Freenom. Watch how to do that by clicking on these links below

  https://www.youtube.com/watch?v=hRSj2n-XKGM

  https://www.youtube.com/watch?v=IjcHp94Hq8A

  ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ae2ede73-03ce-4af1-9c2e-575182b847bf)

NOTE : As you proceed with configuration, ensure that all resources are appropriately tagged, for example:

- Project:<Give your project a name>

- Environment:<dev>

- Automated: (If you create a recource using an automation tool, it would be )
 
 # Setting Up Infrastucture

 1. Create a VPC

 ![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c3fa559b-8e4e-4aaa-aaa4-13605712f062)

 2. Create subnets as shown in the architecture

 ![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8a212d26-1108-4997-bd1f-57287fb1a104)

 3. Create a route table for the public subnets

 4. Create a route table for the private subnets

 5. Create an Internet Gateway and attach it to VPC

 6. Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

 7. Edit routes of both public and private route-table and add Intenet Gateway and Nat Gateway to them

 8. Edit Subnet Associations of both Public and Private route-table and associate public and private subnets to them

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/851ea247-655d-46bc-ac31-69f933a3f34f)

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c8bd70ca-98bf-4ecd-9786-d2d022df766a)

9. Create a Security Group for:
    
 - Application Load Balancer: ALB will be available from the Internet.

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/19474dad-1af8-4337-907d-4faa588a7e20)

10. - Bastion Servers:  Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/bf10959f-cad5-4713-a02b-428fc4fe3d33)

11. - Nginx Servers: Access to Nginx should only be allowed from a Application Load balancer (ALB). Alt text
   
      ![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/df914b40-0369-4ead-a97f-9d2e10da33a4)

12. - Internal Application Load Balancer: ALB will be available from the Nginx servers.

    ![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a123b3a4-4a6c-45ba-b94b-3a101d4caa30)

13. - Webservers: Access to Webservers should only be allowed from the Internal Application Load Balancer. Alt text

      ![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/41c90be8-c4f9-4a7f-8a76-cf732c619f89)

14. - Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

    ![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0f720e4e-29f4-4020-b1b5-6272e20c7908)


# TLS Certificates From Amazon Certificate Manager (ACM)

You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

- Navigate to AWS ACM

- Request a public wildcard certificate for the domain name you registered.

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/610e3f71-0888-45db-bfaa-c70dc107b922)

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/48915dbc-ccf7-4846-aa00-c5d922526385)

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7aa8c287-e8cb-4ebf-b141-7a0f7fc63c7f)

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fe7eb70c-6fb9-42bc-9754-3e13eb9c614c)

![Snipe 16](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/08c638a3-4e8d-43bb-b12a-975de9a3d278)

![Acm](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/31d0dacb-f196-46b7-9099-d1e668b394d5)

# Configure EFS

- Create a new EFS File system. Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer. Associate the Security groups created earlier for data layer.

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1f51f281-c021-4957-aa4b-0c2d99600517)

![Snipe 18](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5e6aeac0-6aee-4bea-912e-a8206a99f6ed)

![Snipe 19](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f11256a9-1855-4ea5-8098-3e53b4be4f36)

![Snipe 20](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/73118697-2734-4cf3-8c40-c65af6b27a38)

![Snipe 21](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b1f43a8a-00e7-45eb-915f-0a5ccc9aa6e0)

- Create 2 access points - ! for each of the website (wordpress and tooling) so that the files do not overwrite each other when we mount.

  ![Snipe 22](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/741eead1-e039-4d18-915e-4a0489c7a7fb)

  ![Snipe 23](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/83712bb6-3ea0-42b9-b3cd-d87e75a7cf92)

  ![Snipe 24](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1e449109-8bbe-40f9-baed-da81bff02001)

  ![Snipe 25](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/93441a82-a2b5-4e14-8b3f-c4502dc91c15)

   ![Snipe 26](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a3c041b5-4821-442f-8b85-b9582d0a47ed)

  ![Snipe 27](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/53dd6cb1-171a-4543-9e5f-43e399e2b4d1)

  ![Snipe 28](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0adab37e-c818-43dc-84ff-287559d8d5f6)

  ![Snipe 29](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e2aaf969-281c-4f46-afaf-e6ea0163a163)

  # Configure RDS

  # Pre-requisite:

  - Create a KMS Key

   ![Snipe 30](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7a2c2d24-0f53-4ec8-898a-bb7d03561081)

   ![Snipe 31](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/79c5a339-4e45-49ba-b729-dd7a74a57480)

    ![Snipe 32](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c22c1176-29a8-4ac7-bf34-b18aece63df9)

   ![Snipe 33](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e1fa8d98-8c72-4414-b09c-3e76020f17e6)

    ![Snipe 34](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d32d3f64-3b49-4a7b-b6f2-ec4c53cd8ee8)

    ![Snipe 35](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/956fb71b-55e8-44d7-9b65-64578edc2bf0)

To ensure that your databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones.

To configure RDS, follow steps below:

1. Create a subnet group and add 2 private subnets (data Layer)

![Snipe 36](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ab09f704-7bb2-4935-985f-9ab9270b319b)

![Snipe 37](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f4afbd62-67b4-403b-91e5-e3a44ca04bed)

![Snipe 38](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/920c0da2-4ac8-4f4d-9330-c5cd4b1db7d2)

2. Create the DB

![Snipe 39](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/57eb9ed8-dad2-4fb6-b2f6-7941061076ac)

![Snipe 40](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/02a5942c-3650-462c-9da2-f6585e0f85f3)

![Snipe 41](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/075347c0-999d-4713-81b5-4c8e6b763a79)

![Snipe 42](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ff58bd8b-5200-4d84-afc4-9d95b7ce51d8)

![Snipe 43](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b442bc6a-fc7a-4bdb-b615-6d96fb5a7668)

# Configure Loadbalancers and Target Groups

1. Create Target group for NGINX, tooling amd wordpress targets

![Snipe 44](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1ad0a094-71e7-4f30-95db-8fbc9dba68ae)

![Snipe 45](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b5c4d270-c970-4d6d-9726-d3ac595297f7)

![Snipe 46](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/43ec98f5-2ef2-4b21-922c-71355cadb506)

![Snipe 47](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8440c920-7ece-40af-b415-ccfeb2bc9140)

2. Create public-facing and internal loadbalancers

 ![Snipe 48](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c0ec5811-dc7b-4e3a-8b6f-d405381a02d2)
  
![Snipe 49](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ebba69e2-df31-4bea-a91b-7ca5be296ff3)

![Snipe 50](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e0f8545f-e3d2-4264-87d1-901f8aa00aa7)

![Snipe 51](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5d9d2465-a3ab-4225-a4e3-3f7780b78489)

![Snipe 52](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/8e622c1a-7992-4971-86aa-b4679986e46c)

Repeat the same procedure for internal ALB, select wordpress as the default target and create a rule to send traffic to tooling if the headers match our specified parameters.

![hey](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9a72acb3-df5a-4531-949e-dee199f0a991)

![heyy](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d6c7a87e-de1a-42f6-9f16-cdb3d245792b)

![heyyy](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/595a46bd-cca8-4043-9626-6ab77760aec2)

![heyyyy](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4971c725-0c19-4a57-8866-3761f2fbcb78)

![Snipe 53](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f4088084-a950-4abc-8f54-1a4bce8bc335)

![Heyyyyy](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1a8a7a16-a555-4d31-a0fa-27be3c661089)

# SET UP COMPUTE RESOURCES

We will be setting up the AMI for the nginx, bastion and webservers for their various Auto Scaling Groups.

Launch 3 RHEL-8 instances - The AMIs will be used for the ASG. Lauch the instances in the default VPC.

![heyyyyyy](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d3da2723-7b74-4892-a4d7-7f26c89c6401)

# For Bastion Host

Run the following commands

$ sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y

$ sudo systemctl start chronyd

$ sudo systemctl enable chronyd

$ sudo systemctl status chronyd

![Snipe 54](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/08a3eb17-5ce3-451c-a5e2-b4bef56dc22d)

![Snipe 55](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9f5704cd-2eaf-43d8-93f6-303b73fb73e4)

![Snipe 56](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a1d31bb7-4992-40e8-b886-9de52b96417f)

# For Nginx

Run the followinf commands

$ sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y

$ sudo systemctl start chronyd

$ sudo systemctl enable chronyd

$ sudo systemctl status chronyd

# Configure Selinux policies

$ sudo setsebool -P httpd_can_network_connect=1

$ sudo setsebool -P httpd_can_network_connect_db=1

$ sudo setsebool -P httpd_execmem=1

$ sudo setsebool -P httpd_use_nfs 1

![Snipe 57](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1c1c2eb2-2e08-4d0e-84e3-dc8db83183f8)

# Install amazon efs utils for mounting the target on the Elastic file system

$ git clone https://github.com/aws/efs-utils

$ cd efs-utils

$ sudo yum install -y make

$ sudo yum install -y rpm-build

$ sudo make rpm

$ sudo yum install -y  ./build/amazon-efs-utils*rpm

![Snipe 58](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5a8c41ef-35b0-4738-af20-a4ddd268e712)

Setting up self-signed certificate for the nginx instance. If a target group is configured with the HTTPS protocol or uses HTTPS health checks, the TLS connections to the targets use the security settings from the ELBSecurityPolicy-2016-08 policy. The load balancer establishes TLS connections with the targets using certificates that you install on the targets. The load balancer does not validate these certificates. Therefore, you can use self-signed certificates or certificates that have expired. Because the load balancer is in a virtual private cloud (VPC), traffic between the load balancer and the targets is authenticated at the packet level, so it is not at risk of man-in-the-middle attacks or spoofing even if the certificates on the targets are not valid.

$ sudo mkdir /etc/ssl/private

$ sudo chmod 700 /etc/ssl/private

$ sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/narbyd.key -out /etc/ssl/certs/mirahkeys.crt

$ sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048

![Snipe 59](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f0e158c5-07e8-4d68-9995-73844bdd7b0e)

To confirm my cert installation is successful and present in the server

$ sudo ls -l /etc/ssl/certs/

$ sudo ls -l /etc/ssl/private/

Start and enable nginx

# For Webserver

Run the following commands to configure the webserver instance.

$ sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

$ sudo yum install -y dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

$ sudo yum install wget vim python3 telnet htop git mysql net-tools chrony -y

$ sudo systemctl start chronyd

$ sudo systemctl enable chronyd

$ sudo systemctl status chronyd

Configure Selinux policies

$ sudo setsebool -P httpd_can_network_connect=1

$ sudo setsebool -P httpd_can_network_connect_db=1

$ sudo setsebool -P httpd_execmem=1

$ sudo setsebool -P httpd_use_nfs 1

Install amazon efs utils for mounting the target on the Elastic file system

$ git clone https://github.com/aws/efs-utils

$ cd efs-utils

$ sudo yum install -y make

$ sudo yum install -y rpm-build

$ sudo make rpm

$ sudo yum install -y  ./build/amazon-efs-utils*rpm

setting up self-signed certificate for the apache webserver instance

$ sudo yum install -y mod_ssl

$ sudo openssl req -newkey rsa:2048 -nodes -keyout /etc/pki/tls/private/narbyd.key -x509 -days 365 -out /etc/pki/tls/certs/mirahkeys.crt

Using vi editor to edit the SSL certificate file path from localhost.crt and localhost.key to narbyd.crt and narbyd.key respectively

$ sudo vi /etc/httpd/conf.d/ssl.conf

![Snipe 60](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/a632116e-20d4-41d3-b2a5-9ace5e7414a1)

# Create AMIs from the instances

![Snipe 61](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5e589155-e51a-4762-afd4-3961047d96e9)

![Snipe 62](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/979fbffc-aa4b-43ca-b060-4a715fc1d655)

![Snipe 63](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b028ea38-2ee5-4632-8c53-72aad6f47707)

# Create Launch Templates

From the created custom AMIs, create Launch templates for each of the instances

![Snipe 64](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/211af2ac-ff85-4402-a7de-78805d542b05)

![Snipe 65](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3226e21d-a59b-46ce-bf00-05f04c84ff4e)

![Snipe 66](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4d0d9595-c357-4261-bd14-0f92e9d3118d)

![Snipe 67](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b8300159-7ec6-41bd-8b39-34b4e530be3a)

![Snipe 68](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cacd13f3-84dd-42e6-a3c1-4883f5febf9b)

Fill in the userdata for Bastion

#!/bin/bash

yum install -y mysql

yum install -y git tmux

yum install -y ansible

![Snipe 69](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5129cb46-2f13-477b-aece-705c967899b2)

![Snipe 70](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/d8f3bf53-1cf9-47d0-aa2d-9a3768e5a150)

Fill in the userdata for Nginx

#!/bin/bash
yum install -y nginx
systemctl start nginx
systemctl enable nginx
git clone https://github.com/IwunzeGE/wakabetter-project-config.git
mv /wakabetter-project-config/reverse.conf /etc/nginx/
mv /etc/nginx/nginx.conf /etc/nginx/nginx.conf-distro
cd /etc/nginx/
touch nginx.conf
sed -n 'w nginx.conf' reverse.conf
systemctl restart nginx
rm -rf reverse.conf
rm -rf /wakabetter-project-config

Fill in the userdata for Tooling

#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-087db5a73c7b8725b fs-0cf945c5a63141f5b:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
git clone https://github.com/IwunzeGE/DevopsToolingWebsite.git
mkdir /var/www/html
cp -R /DevopsToolingWebsite/html/*  /var/www/html/
cd /DevopsToolingWebsite
mysql -h zhikbee-rds.cg22cjxlsbrj.us-east-2.rds.amazonaws.com -u admin -p toolingdb < tooling-db.sql
cd /var/www/html/
touch healthstatus
sed -i "s/$db = mysqli_connect('mysql.tooling.svc.cluster.local', 'admin', 'admin', 'tooling');/$db = mysqli_connect('zhikbee-rds.cg22cjxlsbrj.us-east-2.rds.amazonaws.com', 'admin', 'password', 'toolingdb');/g" functions.php
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

Fill in the userdata for Wordpress

#!/bin/bash
mkdir /var/www/
sudo mount -t efs -o tls,accesspoint=fsap-028b8045c50f7e635 fs-0cf945c5a63141f5b:/ /var/www/
yum install -y httpd 
systemctl start httpd
systemctl enable httpd
yum module reset php -y
yum module enable php:remi-7.4 -y
yum install -y php php-common php-mbstring php-opcache php-intl php-xml php-gd php-curl php-mysqlnd php-fpm php-json
systemctl start php-fpm
systemctl enable php-fpm
wget http://wordpress.org/latest.tar.gz
tar xzvf latest.tar.gz
rm -rf latest.tar.gz
cp wordpress/wp-config-sample.php wordpress/wp-config.php
mkdir /var/www/html/
cp -R /home/ec2-user/wordpress/* /var/www/html/
cd /var/www/html/
touch healthstatus
sed -i "s/localhost/zhikbee-rds.cg22cjxlsbrj.us-east-2.rds.amazonaws.com/g" wp-config.php 
sed -i "s/username_here/admin/g" wp-config.php 
sed -i "s/password_here/password/g" wp-config.php 
sed -i "s/database_name_here/wordpressdb/g" wp-config.php 
chcon -t httpd_sys_rw_content_t /var/www/html/ -R
systemctl restart httpd

Before creating the Auto Scaling Group for the webservers, we will go into the RDS and create the wordpress database

![Snipe 79](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/98d27c3a-35dd-4024-b9d6-ec80c9e20123)

# Create Auto Scaling Group

For Bastion

![Snipe 71](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1d44a92f-c7fe-45a6-983c-aae833549f09)

![Snipe 72](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e0f0db95-8ab2-46b0-bedc-e0fc6b4cb93d)

![Snipe 73](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e80d5300-cdda-48a4-a38b-a7a6c31c4a4d)

![Snipe 74](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/df233bb8-7bdb-45db-8b7d-a23473c20b14)

For Nginx

![Snipe 75](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/010768ef-b3df-4eb9-926c-9223c1cd24fe)

![Snipe 76](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cf093b05-f19f-4cf3-afe5-dbf08fe9df69)

![Snipe 77](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/17edf578-e8a0-41b0-adcc-1ab3c79b68f9)

![Snipe 78](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/108da242-a885-4457-a30c-25634e96b8bd)

Open the browser incognito using "CTRL + SHIFT + n" and paste the domain name i.e tooling.mirahkeys.xyz



































































































































































































  













  
