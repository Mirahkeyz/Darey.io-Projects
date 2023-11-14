# LEMP STACK IMPLEMENTATION

A Lemp Stack is an application that makes use of NGINX as the Web server for hosting web application.

# STEP 1: Launching an EC2 Instance

Login to AWS Cloud Service console and create an Ubuntu EC2 instance, including the key pair and security groups (TCP = SSH-22, HTTP-80)

![Snipe](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cf939d76-4034-4b5c-b569-f857c74ecd1e)

Login into the instance via ssh:

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/cebd1fac-574d-47c0-99ed-1ec2e521f8e9)

# STEP 2: Installing Nginx

Run a sudo apt update to download package information from all configured sources.
Next, Run sudo apt install nginx 
![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/65f4c64e-5c55-4e23-a6b4-1757d50c0aa5)

After it has installed, Run sudo systemctl status nginx to confirm that nginx is running

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/6b0966b5-4923-4b8b-a81c-53c4747abf93)

To access our server locally on the ubuntu shell, Run curl://http:localhost:80 or curl://http:127.0.0.1:80 

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/edc7dbb0-2330-4ffa-806f-c4b9450c202e)

To check if we can access the default server block over the internet on our local machine, insert the public IP address of the server on a browser.

![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3c2d6023-9786-4505-8d40-5bb0f995a9b2)

# STEP 3: Installing MySql

To install mysql on the ubuntu shell Run sudo apt install mysql-server

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/0e510cad-c990-494d-b6b4-5025ed244f78)
