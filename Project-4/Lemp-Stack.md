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

When the installation is finished, log into the mysql console by typing:
sudo mysql

![Snipe 7](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/246c88da-75c9-49fb-9e73-739430dbbac2)

To create a root password run the code below
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'PassWord.1';
Then exit the mysql shell

Next start the interactive script by typing 

 sudo mysql_secure_installation

![Snipe 8](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/de2842bc-702f-46b4-9c40-8b65a3c3eafa)


# STEP 4: Installing PHP And Its Modules

To install PHP run:
 sudo apt install php-fpm php-mysql

![Snipe 9](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/5984769a-bfc7-4ac1-b119-01e0fd8e0580)

Creating a Web Server Block For our Web Application:

To serve our webcontent on our webserver, we create a directory for our project inside the /var/www/ directory.
sudo mkdir /var/www/projectLEMP Then we change permissions of the projectLEMP directory to the current user system by running sudo chown -R $USER:$USER /var/www/projectLEMP

![Snipe 11](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/ff377e6c-df17-4699-8e7d-547fa8e99436)

Creating a configuration for our server block:
sudo nano /etc/nginx/sites-available/projectLEMP

The following snippets represents the configuration required for our web server block to be functional

#/etc/nginx/sites-available/projectLEMP

server {
    listen 80;
    server_name projectLEMP www.projectLEMP;
    root /var/www/projectLEMP;

    index index.html index.htm index.php;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
     }

    location ~ /\.ht {
        deny all;
    }

}


Then we link the configuration file to the sites-enabled directory

sudo ln -s /etc/nginx/sites-available/projectLEMP /etc/nginx/sites-enabled

To test our configuration for errors we run
sudo nginx -t

![Snipe 12](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/1b5999e5-9fb8-4472-b08a-50b06261f0c9)


