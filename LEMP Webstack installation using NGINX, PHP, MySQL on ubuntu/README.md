<h1> Setting up a LEMP Stack on AWS EC2 (Ubuntu) using DevOps Practices </h1>
<br>  </br>

<h3> This guide explains how to set up a LEMP (Linux, NGINX, MySQL, PHP) stack on an AWS EC2 instance running Ubuntu. By following these instructions, you will have a fully functional web server capable of running PHP applications. </h3>


Step 1: Install NGINX Server

To install NGINX, update the packages and then install NGINX using the following commands:

    sudo apt update
    sudo apt install nginx

After installing NGINX, check its status with:

    sudo systemctl status nginx

You can test the installation locally by visiting localhost:80, or by using the public IP address of your EC2 instance.

To retrieve the Public IP address, you can use the following command:

    curl -s http://169.254.169.254/latest/meta-data/public-ipv4

Step 2: Install MySQL

MySQL is a relational database management system commonly used alongside PHP. 
To install MySQL, run the following command:

    sudo apt install mysql-server

After the installation, log in to MySQL by running:

    sudo mysql

It is recommended to run the security script that comes pre-installed with MySQL. Before running the script, set a password for the root user using mysql_native_password as the default authentication method. For example:

    ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Pa55W0rd..1';

Exit the SQL shell with:

    exit

Start an interactive shell with:

    sudo mysql_secure_installation

You will be prompted to configure the VALIDATE PASSWORD PLUGIN, and in this case, it is disabled while other prompted options are enabled using the Y (Yes) option.

Step 3: Install PHP

After installing NGINX and MySQL, install PHP using the following command:

    sudo apt install php-fpm php-mysql

Configure NGINX to Use the PHP Processor

By default, NGINX has one server block enabled and is configured to serve documents out of the /var/www/html directory. To set up a separate directory for your project, follow these steps:

- Create a root directory for the project. For example:

        sudo mkdir /var/www/LEMP-Stack

- Assign ownership of the directory to the appropriate user. For example:

        sudo chown -R $USER:$USER /var/www/LEMP-Stack

- Create a configuration file for the project. For example:

        sudo nano /etc/nginx/sites-available/LEMP-Stack

Paste the provided code into the empty file and modify the server_name and root values to suit your needs:

    #/etc/nginx/sites-available/LEMP-Stack
    
    server {
    listen 80;
    server_name LEMP-Stack www.LEMP-Stack;
    root /var/www/LEMP-Stack;
    
    index index.html index.htm index.php;
    
    location / {
      try_files $uri $uri/ =404;
    }
    
    location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }
    
    location ~ /\.ht {
      deny all;
    }
    
    }
    
    
    
    

Enable the configuration file by creating a symbolic link in the sites-enabled directory. For example:



    sudo ln -s /etc/nginx/sites-available/LEMP-Stack /etc/nginx/sites-enabled/

Test the configuration with:

    sudo nginx -t

Ensure that the output confirms a successful configuration.

Disable the default NGINX host configured for our new service to run:

    sudo unlink /etc/nginx/sites-enabled/default

Reload NGINX for the changes to take effect:

    sudo systemctl reload nginx


The /var/www/LEMP-Stack directory is now empty, and you can create an index.html file to test it:

    sudo echo 'Hello LEMP-Stack from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP' $(curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/LEMP-Stack/index.html

Test PHP with NGINX

Create an index.php file:

    sudo nano /var/www/LEMP-Stack/info.php

Paste the following PHP code into the file:

    php
    
    <?php
    phpinfo();

Save and close the file.

You can now test the PHP setup by visiting http://<server_domain_or_IP>/info.php in your web browser.

Remember to remove the info.php file or restrict access to it once you have completed the testing.


<h3> Result: </h3>

<img width="1440" alt="Screenshot 2023-07-02 at 22 03 08" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/dd46e327-fc4f-4535-8c5f-33f953c03b1c">




