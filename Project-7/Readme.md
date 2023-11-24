# Implementing LoadBalancers With Nginx

Loadbalancing means distributing works or loads among several servers so that none of the servers get overloaded with work. In this Project we are going to use Nginx because Nginx is a volatile software which can act as a Webserver, Reverse proxy and a Loadbalancer.

# STEP 1: Setting up our instances

We are going to be Launching two EC2 Instances running ubuntu and installing Apache webserver on them, we will edit their inbound rules and add port 8000 to allow traffic from anywhere and then update the default page of the webservers to display their public ip address.

![Snipe 1](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/da429e2e-251e-4609-b9b5-81d7743181f0)

# STEP 2: Install Apache

After launching your EC2 instances then copy their SSH links and go and open two terminal or gitbash window and connect to them by typing the following commands:

sudo apt update
sudo apt install apache2
sudo systemctl status apache2

![Snipe 2](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/3cafe3de-b776-4d7f-97e1-403c046eee08)

# STEP 4: Configure Apache

- we will start by configuring Apache webserver to serve content on port 8000 instead of its default port which is port 80
  Type sudo vi /etc/apache2/ports.conf
  when it opens add a new listen directive for port 8000

![Snipe 3](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/604ddc1c-dd62-4e34-b074-7e07e7e641a6)

- Next open the file /etc/apache2/sites-available/000-default.conf and change port 80 on the virtualhost to 8000

sudo vi /etc/apache2/sites-available/000-default.conf

![Snipe 4](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/7c6c5fab-8353-40f6-b16b-b4f92039074f)

- Restart apache to load the new configuration

sudo systemctl restart apache2

- Create a new index.html file

  sudo vi index.html

Then type the below code and edit were necessary

        <!DOCTYPE html>
        <html>
        <head>
            <title>My EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>

   ![Snipe 5](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/b621a000-9a36-463c-ba71-69195c4f777e)

- Change the ownership of the index.html file

  sudo chown www-data:www-data ./index.html

- Override the Default html file of Apache webserver

sudo cp -f ./index.html /var/www/html/index.html

- Restart the webserver to load the configuration

sudo systemctl restart apache2

![Snipe 6](https://github.com/Mirahkeyz/Darey.io-Projects/assets/134533695/56ac2436-c358-4ab6-94a1-129f22d0e2e3)
