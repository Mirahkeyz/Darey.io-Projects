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

# STEP 4: INSTALLING PHP AND ITS MODULES

We need to install php alongside its modules, php-mysql which is php module that allows php to communicate with the mysql database, libapache2-mod-php which ensures that the apache web server handles the php contents properly.

$ sudo apt install php php-mysql libapache2-mod-php

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e52a0bea-669f-41ba-aef2-f5b83eae4a6d)

On successfull installation of php and its modules we can check the version to see if it was properly installed.

$ php -v

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/55c4104f-e3a2-4d98-95d5-06a95c25a208)

At this point our Lamp Stack is completely installed and fully operational

# STEP 5: DEPLOYING OUR WEBSITE ON APACHE'S VIRTUAL HOST

We will use Apache's virtual hosting feature to set up a virtual host so that we can deploy our web content on the web server. Apache's virtual hosting allows multiple websites to run on the same web server using different IP addresses.

First of all we will create a domain and give it a domain of our choice. i will call mine projectlamp

Create the projectlamp directory using the 'mkdir' command as follows:

$ sudo mkdir /var/www/projectlamp

Next, we assign ownership of the directory using $USER environment variable

$ sudo chown -R $USER:$USER /var/www/projectlamp

![Snipe 13](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/4781c33f-1756-4207-82b7-3e4919f95ee6)

Then create and open a new configuration file in Apache's sites-available directory using the vim editor

$ sudo vi /etc/apache2/sites-available/projectlamp.conf

![Snipe 14](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7cc5fbbc-5450-4db2-a9ba-bf7c0bd60deb)

Run esc :wq  to save and terminate vi editor.

Run sudo a2ensite projectlampstack to activate the server block.

Run sudo a2dissite 000-default to deactivate the default webserver block that comes with apache on default.

Reload the apache2 server sudo systemctl reload apache2

![Snipe 15](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2b6a8f61-359e-445c-a5e0-2e01d80717c4)

Create an index.html file inside the /var/www/projectlampstack

sudo echo 'Hello LAMP from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/projectlamp/index.html

![Snipe 17](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/fcafdace-399f-4d31-b7e8-50007589e7d3)

Now go to your browser and try to open your website URL using the IP address:

http://<Public-IP-Address>:80

if you see the text from 'echo' command you wrote to index.html file, then it means your Apache virtual host is working as expected.





