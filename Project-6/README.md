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
