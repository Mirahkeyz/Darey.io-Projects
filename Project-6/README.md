# IMPLEMENTING CLIENT SERVER ARCHITECTURE USING MYSQL RELATIONAL DATABASE MANAGEMENT SYSTEM DBMS

Client-Server refers to an architecture in which two or more computers are connected together over a network to send and receive requests between one another. In their communication, each machine has its own role: the machine sending requests is usually referred as "Client" and the machine responding (serving) is called "Server".

To set up a Client-Server Architecture with Mysql using EC2, we need two instances. We will name them:
- Mysql client.
- Msql server.

 We set this up by doing the following:
- create an account on AWS.
- we create two instances by selecting “ubuntu server 20.04 LTS” from Amazon Machine Image(AMI)(free tier).
- we select “t2.micro(free tier eligible)”.
- then go to the security group and select “a security group” review and launch.
  ![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1fb257ab-ce3b-41b4-a7c0-92fd315de733)

We open our terminal or Git bashand go to the location of the previously downloaded PEM file.
We connect to the instances from two seperate ubuntu terminal or Git bash

After connecting to the instances on each of the terminals, we edit the /etc/hostname/ file to change the names of each of the server so as to align with the given name on the instance i.e client and server respectively. We do this using the command:

sudo su
then
sudo chmod u+w /etc/hostname
then
sudo vim /etc/hostname

This opens the hostname file. We then edit the content to suit the given names client and server respectively. We press ESC :wq and ENTER to save.
We then run the commands on the client and server terminals respectively.

$ hostname client for the client
and
$ hostname server for the server.
we disconnect and reconnect to the instances for these changes to take effect.

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/2ffeb247-f314-4b5d-af78-e62f44f3d63d)

On the client and server terminals we run the command:

sudo apt update

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7658e7fa-c2dd-4a55-869e-74a9f9a37d6e)


On the server terminal, we install the mysql server.

sudo apt install mysql-server

and on the client terminal, we install the mysql client

sudo apt install mysql-client

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/36068971-e6b8-4433-aba3-5276275d0595)

We verify the status of mysql by running the command:

sudo systemctl status mysql

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/9b22d20a-caad-4687-9f2e-eb30b31db4cb)


By default, both of the EC2 virtual servers are located in the same local virtual network, so they can communicate to each other using local IP addresses. MySQL server uses TCP port 3306 by default, so you will have to open it by creating a new entry in ‘Inbound rules’ in ‘mysql server’ Security Groups. For extra security, do not allow all IP addresses to reach your ‘mysql server’ – allow access only to the specific local IP address of your ‘mysql client’. take ur mysql client private ip and add it as you are editing the inbound rules of mysql server

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/c76e4369-521a-4f54-9398-8006e10c7e3a)


We need to configure MySQL server to allow connections from remote hosts.

sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

then

Replace "bind-address" ‘127.0.0.1’ to ‘0.0.0.0’ 

comment mysqlx-bind-address like this # mysqlx-bind-address

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/920e5d0b-b37b-44db-9d14-b341eb27deea)


To set up the mysql database in server that the client will be able to connect to, we run the following commands:

$ sudo mysql

this lauches us into the mysql database.

Next we run the command:

mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';

we exit. Start the interactive script by running:

$ sudo mysql_secure_installation

At the prompt we put in the password we specified earlier i.e PassWord.1. To create a validated password, type y. Then choose the strength of the new password you want to create-at the prompt, we put in the new password.

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/e82fc4eb-2502-4948-9381-ce962f0ad066)

