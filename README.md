# Linux Server Configuration Project
>By Abdirashiid Jama

### Description
the objective of this project is to take a baseline installation of a Linux server and prepare it to host web applications. The student will secure the server from a number of attack vectors, install and configure a database server, and deploy one of my existing web applications onto it.

* SSH port: 2200
* IP address: 52.56.160.20
* Grader password if asked will be sent with project comments

## Creating a Ubuntu instance on AWS
1. [Sign up with AWS](https://lightsail.aws.amazon.com)
2. Click "Create a Instance"
3. Choose "OS Only"
4. Then choose "Ubuntu 16.04 LTS"
5. Select a instance plant (Lowest in my case) and name it
6. Proceed to create it

## Accessing the Server
1. Download your private key from the "Account section" it's should be names something like "LightsailDefaultKey-eu-west-2.pem"
2. Create a new file named "aws_key.rsa " under ~/.ssh folder on your local machine (If you do not have a .ssh folder, create it)
3. Open the downloaded key like `cat LightsailDefaultKey-eu-west-2.pem` and copy the content
3. paste the content into  "aws_key.rsa"
4. Set the file permission to owner only : `chmod 600 ~/.ssh/aws_key.rsa`
5. Log in to the server like: `ssh -i ~/.ssh/aws_key.rsa ubuntu@52.56.160.20`

## Updating the current packages
1. type `sudo apt-get update` to update the packages
2. type `sudo apt-get upgrade` to install the latest versions

##  Changing the SSH port from 22 to 2200
1. type `sudo nano /etc/ssh/sshd_config` 
2. Change the port number from "22" to "2200" (Should be near the top)
3. Save and exit the file
4. type `sudo service ssh restart` to restart SSH

## Readying the firewall
1. Start by Checking the firewall status by typing: `sudo ufw status`
2. Change default firewall to deny all incoming: `sudo ufw default deny incoming`
3. Change default firewall to allow all outgoing: `sudo ufw default allow outgoing`
6. Allow incoming UDP packets on port 123 to allow NTP: `sudo ufw allow 123/udp`
4. Allow incoming TCP on port 2200 to allow SSH: `sudo ufw allow 2200/tcp`
6. Allow incoming TCP on port 80 to allow www: `sudo ufw allow www`
7. Close port 22 for security: `sudo ufw deny 22`
8. Enable the firewall: `sudo ufw enable`
9. Check the firewall status: `sudo ufw status`
10. Be sure too add "TCP port:2200" and "UDP port:123" to the firewall on AWS under networking and Delete "port:22"
11. ssh in using the new port 2200: `ssh -i ~/.ssh/aws_key.rsa ubuntu@52.56.160.20 -p 2200`

## Creating a new user called "grader" and giving it sudo access
1. Create a new user called "grader" by typing:`sudo adduser grader`
2. type `sudo nano /etc/sudoers`
3. Create a file named grader under: `sudo touch /etc/sudoers.d/grader`
4. Edit it to give "grade" sudo access: `sudo nano /etc/sudoers.d/grader`, add code `grader ALL=(ALL:ALL) ALL`. 

## Logging in with the SSH key pairs
1. Create an SSH key pair for "grader" by typing `ssh-keygen` on your local machine. when it prompts for a location save it in `~/.ssh` 
2. to move your public key the server go to your local machine, open the public key by typing `cat ~/.ssh/[name of file].pub`
3. On your server type `mkdir .ssh` to create a .ssh folder
4. create a "authorized_keys" file by typing `touch .ssh/authorized_keys`
5. then type `nano .ssh/authorized_keys` to edit the file and copy the public key to this file
6. type `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` to restrict permissions
7. Restart SSH by typing: `sudo service ssh restart`
8. log in as grader by typing: `ssh -i ~/.ssh/grader_key -p 2200 grader@52.56.160.20`
9. for security reason to disable password type `sudo nano /etc/ssh/sshd_config`
10. Change the line `PasswordAuthentication yes` to a "no"
11. Restart SSH by typing: `sudo service ssh restart`

## Changing the local timezone to UTC
1. Type `sudo dpkg-reconfigure tzdata`
2. choose "none of the above" and then choose "UTC"

## Installing and configuring Apache
1. Install "apache2" by typing: `sudo apt-get install apache2`
2. Go to http://52.56.160.20/, apache will show a default page if it is working

## Installing and configuring Python mod_wsgi
1. Install "mod_wsgi" package by typing: `sudo apt-get install libapache2-mod-wsgi python-dev`
2. Then enable it: `sudo a2enmod wsgi`
3. Restart the "Apache2" service by typing: `sudo service apache2 restart`

## Installing PostgreSQL
1. type `sudo apt-get install postgresql` to install PostgreSQL

## Creating a PostgreSQL user called "Catalog"
1. Access the PostgreSQL default user "postgres" by typing: `sudo su - postgres`
2. Connect to PostgreSQL by typing: `psql`
3. Create a user names "catalog" with LOGIN role: `# CREATE ROLE catalog WITH PASSWORD 'randompass';`
4. Allow the user to create database tables: `# ALTER USER catalog CREATEDB;`
5. Create a database: `# CREATE DATABASE catalog WITH OWNER catalog;`
6. Connect to the database **catalog**: `# \c catalog`
7. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
8. Grant access to the "catalog" user: `# GRANT ALL ON SCHEMA public TO catalog;`
9. Exit psql by typing: `\q`
10. `^d` to exit psql

## Creating a Linux user names "catalog" and a database
1. Create a new user by typing: `sudo adduser catalog`
2. Give the "catalog" user sudo access through: `sudo visudo`
3. Copy the folowing into it: `catalog ALL=(ALL:ALL) ALL` under line `root ALL=(ALL:ALL) ALL`
4. Log in as the "catalog" user: `sudo su - catalog`
5. Create a database called "catalog": `createdb catalog`
6. Exit by `^d`

## Installing git and cloning the catalog project via Github
1. type `sudo apt-get install git`
2. Clone the project: `sudo git clone https://github.com/Rashiid-Dev/Item-Catalog`
5. Change the the owner of the folder: `sudo chown -R ubuntu:ubuntu catalog/`
6. CD to `/var/www/Item-Catalog/catalog`
7. Change `application.py` to `__init__.py` by typing: `mv application.py __init__.py`
8. Change line `app.run(host="0.0.0.0", port=5000, debug=False)` to `app.run()` in the `__init__.py` file

## Readying the server for the Flask project
1. Install pip by typing: `sudo apt-get install python-pip`
2. Installing the packages:
In my case I need to add the -H flag to sudo like this `sudo -H install (packagename)
* Flask==1.0.2
* httplib2==0.11.3
* oauth2client==4.1.3
* psycopg2==2.7.5
* SQLAlchemy==1.2.13
* requests

## Setting up and enabling the virtual host
1. Create a file by typing: `sudo touch /etc/apache2/sites-available/catalog.conf`
2. Add this to the file:
`
<VirtualHost *:80>
                ServerName 52.56.160.20
                ServerAdmin grader@52.56.160.20
                WSGIDaemonProcess application  python-path=/home/grader/.local/lib/python2.7/site-packages
                WSGIProcessGroup application
                WSGIScriptAlias / /var/www/Item-Catalog/catalog.wsgi
                <Directory /var/www/Item-Catalog/catalog/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                Alias /static /var/www/Item-Catalog/catalog/static
                <Directory /var/www/Item-Catalog/catalog/static/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
`
3. type `$ sudo a2ensite catalog` to start the virtual host (It might already be enabled)
4. Restart the "apache2" service: `sudo service apache2 reload`

## Configuring the .wsgi file
1. Create this file by typing: `sudo touch /var/www/Item-Catalog/catalog.wsgi`
2. Add the folloing to the file:

`import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Item-Catalog/catalog")
from __init__ import app as application
application.secret_key = 'Testkey'`

3. Restart the "apache2" service: `sudo service apache2 reload`

## Editing the database paths
1. Replace lines in `__init__.py`, `injector.py`, and `database_setup.py` files with `engine = create_engine('postgresql://catalog:randompass@localhost/catalog')`

## Disabling the default Apache2 page
1. type: `a2dissite 000-default.conf`
2. Restart the "apache2" service: `sudo service apache2 reload`

## Setting up the database
1. While in /var/www/Item-Catalog/catalog type: `sudo python database_setup.py` to create the database
2. and ` sudo python injector.py` to populate the database
3. Restart the "apache2" service: `sudo service apache2 reload`
4. go to  http://52.56.160.20/ and you should see the website up and running












