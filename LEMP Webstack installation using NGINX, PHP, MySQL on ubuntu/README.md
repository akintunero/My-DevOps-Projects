<h1> Setting up a LEMP stack </h1>
<br>  </br>
Step 1: Install the NGINX server
To install NGINX, update the packages and then install NGINX using the following commands:

    sudo apt update
    sudo apt install nginx
    
After installing NGINX, check its status with:

    sudo systemctl status nginx
    

You can test the installation locally by visiting localhost:80, or by visiting http://<Public-IP-Address>:80.

To retrieve the Public IP address, you can use the following command:
  
     curl -s http://169.254.169.254/latest/meta-data/public-ipv4
  
Step 2: Install MySQL
MySQL is a relational database management system usually used alongside PHP.
  
To install MySQL, run the following command:
  
     sudo apt install mysql-server
  
  
After the installation, log in to MySQL by running:
  
      sudo mysql
  
 Its recommend running the security script that comes pre-installed with MySQL. Before running the script, set a password for the root user using mysql_native_password as the default authentication method. For example:

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
By default, Nginx has one server block enabled and is configured to serve documents out of the /var/www/html directory. To set up a separate directory for your project, follow these steps:

        Create a root directory for the project. For example:
                
                sudo mkdir /var/www/LEMP-Stack

        Assign ownership of the directory using the $USER environmental variable that references the current user. For example:
                
                sudo chown -R $USER:$USER /var/www/LEMP-Stack

        Create a configuration file using Nginx's bare bones configuration. For example:
  
                 sudo nano /etc/nginx/sites-available/LEMP-Stack

        Paste the following code into the empty file and modify the server_name and root values to suit your needs:
  
  
  
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

and the feedback below confirms installation to be OK.

nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful.
  
  
We also need to disable the default NGNIX host configured for our new service to run:

        sudo unlink /etc/nginx/sites-enabled/default

Reload NGINX for effect to take place:

        sudo systemctl reload nginx

The /var/www/LEMP-Stack is empty and we can create an index.html to test with: 

          sudo echo 'Hello LEMP-Stack from hostname' $(curl -s http://169.254.169.254/latest/meta-data/public-hostname) 'with public IP'
          (curl -s http://169.254.169.254/latest/meta-data/public-ipv4) > /var/www/LEMP-Stack/index.html


  
Test PHP with Nginx

Create an index.php file

          sudo nano /var/www/LEMP-Stack/info.php

Paste the php info in the configuration file

            <?php
            phpinfo();

Test the php set with

              http://`server_domain_or_IP`/info.php



