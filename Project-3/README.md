# DEPLOYING A LAMP STACK WEB APPLICATION ON AWS


A Lamp Stack is a popular web stack used for building dynamic websites and web application. It stands for Linux, Apache, Mysql and PHP, Perl, Python.

Linux: The operating system that runs the other components of the LAMP stack.

Apache: The software that delivers web pages to users.

MySQL: The database that stores data for websites.

PHP, Perl, or Python: The programming language that creates dynamic web pages.

# STEP 1: LAUNCHING AN EC2 INSTANCE

First we log on to AWS Cloud Services and create an EC2 Ubuntu VM instance. When creating an instance, choose keypair authentication and download private key(*.pem) on your local computer and Create a security group which has a TCP port of SSH 22 and HTTP 80

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4c06986b-a870-403c-90b3-b678e79bc0a4)

On windows terminal, cd into the directory containing the downloaded private key.Run the below command to log into the instance via ssh:
ssh -i <private_keyfile.pem> username@ip-address

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/f9cd2069-793d-4d59-a71e-b735b00e88cb)

# STEP 2: SETTING UP AN APACHE WEB SERVER

To deploy the web application, we need toinstall apache via ubuntu package manager apt:

$ sudo apt update

$ sudo apt install apache2

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/dded1dfc-33c3-4b95-9183-07a44b7d9e9b)

To check the status of the already installed apache:

$ sudo systemctl status apache2

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7e194791-84d6-4405-b792-90fab9459ece)

To check the accessiblity of our web server on the internet, we curl the IP address/DNS name of our localhost.
Curl http://localhost:80 or Curl http://127.0.0.1:80 

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7af8aa0b-a745-4cd7-8dc8-222542a14104)

To see if our web application server can respond to requests , use the public ip address of our instance on a web browser. http://<Public-IP-Address>:80

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/379ed366-7b93-41f8-954a-e3a9ccee784f)

# STEP 3: INSTALLING MYSQL

Install mysql using the following code:

$ sudo apt install mysql-server

![SNIPE 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b7888a99-2bb8-4c58-8f97-4b8deee0145b)

Use the following command to check if mysql is installed

$ sudo mysql

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e3c4c846-553c-43b5-9a5d-3616b142ecf8)

Use the following command to remove insecure default settings and enable protection for the database.

$ sudo mysql_secure_installation

![Snipe 10](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/59a83404-2647-432d-97de-64273dccd236)

Exit from the MySQL terminal by typing the following command

$ mysql> exit





