# Linux-Configuration
This is the last project toward Udacity Full Stack Web Developer Nanodegree. In this project, the Item Catalog Web Application from will be hosted by a Ubuntu Linux server on an Amazon Lightsail instance. A series of instructions will be presented below. You can visit http://13.233.212.34/ for the website deployed. 


Start a new Ubuntu Linux Server instance on Amazon Lightsail
Create an AWS account
Click Create instance button on the home page
Select Linux/Unix platform
Select OS Only and Ubuntu as blueprint
Select an instance plan
Name your instance
Click Create button
SSH into your Server
Download private key from the SSH keys section in the Account section on Amazon Lightsail. The file name should be like LightsailDefaultPrivateKey-us-east-2.pem
Create a new file named lightsail_key.rsa under ~/.ssh folder on your local machine
Copy and paste content from downloaded private key file to lightsail_key.rsa
Set file permission as owner only : $ chmod 600 ~/.ssh/lightsail_key.rsa
SSH into the instance: $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.218.99.181
Update all currently installed packages
Run sudo apt-get update to update packages
Run sudo apt-get upgrade to install newest versions of packages
Set for future updates: sudo apt-get dist-upgrade
Change the SSH port from 22 to 2200
Run $ sudo nano /etc/ssh/sshd_config to open up the configuration file
Change the port number from 22 to 2200 in this file
Save and exit the file
Restart SSH: $ sudo service ssh restart
Configure the firewall
Check firewall status: $ sudo ufw status
Set default firewall to deny all incomings: $ sudo ufw default deny incoming
Set default firewall to allow all outgoings: $ sudo ufw default allow outgoing
Allow incoming TCP packets on port 2200 to allow SSH: $ sudo ufw allow 2200/tcp
Allow incoming TCP packets on port 80 to allow www: $ sudo ufw allow www
Allow incoming UDP packets on port 123 to allow NTP: $ sudo ufw allow 123/udp
Close port 22: $ sudo ufw deny 22
Enable firewall: $ sudo ufw enable
Check out current firewall status: $ sudo ufw status
Update the firewall configuration on Amazon Lightsail website under Networking. Delete default SSH port 22 and add port 80, 123, 2200
Open up a new terminal and you can now ssh in via the new port 2200: $ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.218.99.181 -p 2200
Create a new user account grader and give grader sudo access
Create a new user account grader:$ sudo adduser grader
$ sudo nano /etc/sudoers
Create a file named grader under this path: $ sudo touch /etc/sudoers.d/grader
Edit this file: $ sudo nano /etc/sudoers.d/grader, add code grader ALL=(ALL:ALL) ALL. Save and exit
Set SSH login using keys
Create an SSH key pair for grader using the ssh-keygen tool on your local machine. Save it in ~/.ssh path
Deploy public key on development environment
On your local machine, read the generated public key cat ~/.ssh/FILE-NAME.pub
On your virtual machine
   $ mkdir .ssh
   $ touch .ssh/authorized_keys
   $ nano .ssh/authorized_keys
Copy the public key to this authorized_keys file on the virtual machine and save
Run chmod 700 .ssh and chmod 644 .ssh/authorized_keys on your virtual machine to change file permission
Restart SSH: $ sudo service ssh restart
Now you are able to login in as grader: $ ssh -i ~/.ssh/grader_key -p 2200 grader@18.218.99.181
You will be asked for grader's password. To unable it, open configuration file again: $ sudo nano /etc/ssh/sshd_config
Change PasswordAuthentication yes to no
Restart SSH: $ sudo service ssh restart
Configure the local timezone to UTC
Run $ sudo dpkg-reconfigure tzdata
Choose None of the above to set timezone to UTC
Install and configure Apache
Install Apache: $ sudo apt-get install apache2
Go to http://18.218.99.181/, if Apache is working correctly, a Apache2 Ubuntu Default Page will show up
Install and configure Python mod_wsgi
Install the mod_wsgi package: $ sudo apt-get install libapache2-mod-wsgi python-dev
Enable mod_wsgi: $ sudo a2enmod wsgi
Restart Apache: $ sudo service apache2 restart
Check if Python is installed: $ python
Install PostgreSQL
Run $ sudo apt-get install postgresql
Make sure PostgreSQL does not allow remote connections
Open file: $ sudo nano /etc/postgresql/9.5/main/pg_hba.conf
Check to make sure it looks like this:
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
Create new PostgreSQL user called catalog
Switch to PostgreSQL defualt user postgres: $ sudo su - postgres
Connect to PostgreSQL: $ psql
Create user catalog with LOGIN role: # CREATE ROLE catalog WITH PASSWORD 'password';
Allow user to create database tables: # ALTER USER catalog CREATEDB;
Create database: # CREATE DATABASE catalog WITH OWNER catalog;
Connect to database catalog: # \c catalog
Revoke all the rights: # REVOKE ALL ON SCHEMA public FROM public;
Grant access to catalog: # GRANT ALL ON SCHEMA public TO catalog;
Exit psql: \q 10.Exit user postgres: exit
Create new Linux user called catalog and new database
Create a new Linux user: $ sudo adduser catalog
Give catalog user sudo access:
$ sudo visudo
Add $ catalog ALL=(ALL:ALL) ALL under line $ root ALL=(ALL:ALL) ALL
Save and exit the file
Log in as catalog: $ sudo su - catalog
Create database catalog: createdb catalog
Exit user catalog: exit
Install git and clone catalog application from github
Run $ sudo apt-get install git
Create dictionary: $ mkdir /var/www/catalog
CD to this directory: $ cd /var/www/catalog
Clone the catalog app: $ sudo git clone RELEVENT-URL catalog
Change the ownership: $ sudo chown -R ubuntu:ubuntu catalog/
CD to /var/www/catalog/catalog
Change file application.py to init.py: $ mv application.py __init__.py
Change line app.run(host='0.0.0.0', port=8000) to app.run() in init.py file
Edit client_secrets.json file
Create a new project on Google API Console and download client_scretes.json file
Copy and paste contents of downloaded client_scretes.json to the file with same name under directory /var/www/catalog/catalog/client_secrets.json
Setup for deploying a Flask App on Ubuntu VPS
Install pip: $ sudo apt-get install python-pip
Install packages:
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
Setup and enble a virtual host
Create file: $ sudo touch /etc/apache2/sites-available/catalog.conf
Add the following to the file:
   <VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin admin@xx.xx.xx.xx
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		<Directory /var/www/catalog/catalog/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
Run $ sudo a2ensite catalog to enable the virtual host
Restart Apache: $ sudo service apache2 reload
Configure .wsgi file
Create file: $ sudo touch /var/www/catalog/catalog.wsgi
Add content below to this file and save:
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/nuevoMexico/")

   from nuevoMexico import app as application
   application.secret_key = 'super_secret_key'
Restart Apache: $ sudo service apache2 reload
Edit the database path
Replace lines in __init__.py, database_setup.py, and lotsofitems.py with engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')
Disable defualt Apache page
$ sudo a2dissite 000-defualt.conf
Restart Apache: $ sudo service apache2 reload
Set up database schema
Run $ sudo python database_setup.py
Run $ sudo python lotsofitems.py
Restart Apache: $ sudo service apache2 reload
Now follow the link to http://18.218.99.181/ the application should be runing online
If internal errors occur: check the Apache error file
Sources
Amazon Lightsail Website
Google API Concole
Udacity
Apache
