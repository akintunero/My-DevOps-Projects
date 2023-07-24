<h2> Setting Up WordPress on RedHat EC2 Instances </h2>

This step-by-step guide will help you set up a DevOps project to deploy WordPress on two RedHat EC2 instances, where one instance acts as a web server and the other as a database server. The web server will use Logical Volume Management (LVM) to manage its storage.

Step 1: Set up the Web Server

Launch two RedHat EC2 instances on AWS. One will act as the web server and the other as the database server.
Create three volumes in the same Availability Zone (AZ) as your web server EC2 instance, each with a size of 10 GiB.
Attach the volumes to your web server EC2 instance.


SSH into the web server instance and run the following command to inspect the attached block devices:
  
    lsblk

Use the df -h command to see all mounted filesystems and free space on your server.

Use the gdisk utility to create a single partition on each of the three disks:
  
    sudo gdisk /dev/xvdf
  
<img width="570" alt="Screenshot 2023-07-23 at 23 26 30" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/739013c3-c405-4946-b536-8d8e1dc8f0da">

Use the lsblk utility to view the newly configured partitions on each of the three disks:
  lsblk
<img width="569" alt="Screenshot 2023-07-23 at 23 28 10" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/52a6d3aa-89db-45ed-a892-4bc5a1c823de">

Install the lvm2 package using:
 
    sudo yum install lvm2

Run the command below to check for available partitions:
 
    sudo lvmdiskscan
  
<img width="571" alt="Screenshot 2023-07-23 at 23 29 09" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/da2795d4-3b24-4a34-bdbd-3d1d63e120f1">

Use the pvcreate utility to mark each of the three disks as physical volumes (PVs) to be used by LVM:
 
    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1

Verify that your physical volumes have been created successfully by running:
  
    sudo pvs
  
  <img width="340" alt="Screenshot 2023-07-24 at 00 25 02" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/785f8192-be6c-43b8-bee8-1e5b67895479">

Use the vgcreate utility to add all three PVs to a volume group (VG) named webdata-vg:
  
    sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1


Verify that your volume group has been created successfully by running:

    sudo vgs
  
<img width="565" alt="Screenshot 2023-07-24 at 00 26 40" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/03438b13-8530-49bc-baa1-4e454aa359f7">

Use the lvcreate utility to create two logical volumes. One named apps-lv (using half of the PV size) and the other named logs-lv (using the remaining space of the PV size):
  
    sudo lvcreate -n apps-lv -L 14G webdata-vg
    sudo lvcreate -n logs-lv -L 14G webdata-vg
  
<img width="570" alt="Screenshot 2023-07-24 at 00 27 54" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/dfff5629-c84c-4f52-ada8-381c6395639f">

Verify that your logical volumes have been created successfully by running:

    sudo lvs
  
<img width="575" alt="Screenshot 2023-07-24 at 00 28 33" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/68816024-8a2c-4712-9038-cffb0516f656">

Verify the entire setup by running:

    sudo vgdisplay -v # View complete setup - VG, PV, and LV
    sudo lsblk
  
<img width="567" alt="Screenshot 2023-07-24 at 00 32 29" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/d9742610-2d5b-4f39-8c6e-e6241906ba98">


Use mkfs.ext4 to format the logical volumes with the ext4 filesystem:

    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

Create the /var/www/html directory to store website files:
  
    sudo mkdir -p /var/www/html

Create the /home/recovery/logs directory to store the backup of log data:
 
    sudo mkdir -p /home/recovery/logs

Mount /var/www/html on the apps-lv logical volume:
 
    sudo mount /dev/webdata-vg/apps-lv /var/www/html/

Use the rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs:

    sudo rsync -av /var/log/. /home/recovery/logs/

Mount /var/log on the logs-lv logical volume. Note that all existing data on /var/log will be deleted, so make sure the backup was successfully
 
    sudo mount /dev/webdata-vg/logs-lv /var/log

Restore log files back into the /var/log directory:
  
    sudo rsync -av /home/recovery/logs/. /var/log

Update the /etc/fstab file so that the mount configuration persists after a restart of the server. Use the UUID of the device to update the /etc/fstab file:
  
    sudo blkid
    sudo vi /etc/fstab

Add entries in the following format (replace UUID and other details with your values):


  UUID=<UUID-of-logical-volume>  /var/www/html  ext4  defaults  0 0
  UUID=<UUID-of-logical-volume>  /var/log       ext4  defaults  0 0


Test the configuration and reload the daemon:
 
    sudo mount -a
    sudo systemctl daemon-reload

Verify your setup by running df -h, and the output should look like this:

<img width="560" alt="Screenshot 2023-07-24 at 00 41 00" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/b0b3fec2-45cc-4ce5-ba23-a373ab1d5495">


Step 2: Prepare the Database Server

Launch a second RedHat EC2 instance that will have the role of 'DB Server.'
Repeat the same steps as in Step 1 (Web Server) for setting up and configuring volumes and LVM on the Database Server, but instead of apps-lv, create db-lv, and mount it to /db directory.

<h2> Step 3: Install WordPress on the Web Server EC2 </h2>

Update the repository and install Apache and its dependencies:

    sudo yum -y update
    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

Start Apache:

    sudo systemctl enable httpd
    sudo systemctl start httpd

Install PHP and its dependencies:

    sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
    sudo yum module list php
    sudo yum module reset php
    sudo yum module enable php:remi-7.4
    sudo yum install php php-opcache php-gd php-curl php-mysqlnd
    sudo systemctl start php-fpm
    sudo systemctl enable php-fpm
    setsebool -P httpd_execmem 1

Restart Apache:

    sudo systemctl restart httpd

Download WordPress and copy it to /var/www/html:

    mkdir wordpress
    cd wordpress
    sudo wget http://wordpress.org/latest.tar.gz
    sudo tar xzvf latest.tar.gz
    sudo rm -rf latest.tar.gz
    cp wordpress/wp-config-sample.php wordpress/wp-config.php
    cp -R wordpress /var/www/html/

Configure SELinux policies:

    sudo chown -R apache:apache /var/www/html/wordpress
    sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
    sudo setsebool -P httpd_can_network_connect=1

Step 4: Install MySQL on your DB Server EC2

Update the repository and install MySQL server:

    sudo yum update
    sudo yum install mysql-server

Verify that the service is up and running:
  
    sudo systemctl status mysqld
  
<img width="571" alt="Screenshot 2023-07-24 at 01 28 08" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/c21e1748-a92f-49e2-a3ee-208bd9455a57">


If it's not running, restart the service and enable it to run on startup:
   
    sudo systemctl restart mysqld
    sudo systemctl enable mysqld
    
<img width="564" alt="Screenshot 2023-07-24 at 01 31 14" src="https://github.com/akintunero/My-DevOps-Projects/assets/13016369/71c649ec-021b-40ed-ac15-c60ea6ae72ab">


Step 5: Configure DB to work with WordPress
    
    sudo mysql

Create a new database for WordPress and a user with access to this database:

    CREATE DATABASE wordpress;
    CREATE USER 'myuser'@'<Web-Server-Private-IP-Address>' IDENTIFIED BY 'password';
    GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
    FLUSH PRIVILEGES;
    SHOW DATABASES;
    exit

Step 6: Configure WordPress to Connect to Remote Database

Install MySQL client on your Web Server:

  sudo yum install mysql

Test the connection to your DB server:


    sudo mysql -u username -p -h <DB-Server-Private-IP-address>

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases.

Change permissions and configuration so Apache can use WordPress.

Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation’s IP).

Access your WordPress installation from your browser using the link:



http://<Your-Web-Server-IP>/wordpress/

Fill out your DB credentials in WordPress setup, and if you see the confirmation message, it means your WordPress has successfully connected
to your remote MySQL database.

![210138663-27322aaf-f020-4261-a695-a98dc5f2387c](https://github.com/akintunero/My-DevOps-Projects/assets/13016369/abff7a21-71ce-4d52-9298-cd4ad1dfe722)



