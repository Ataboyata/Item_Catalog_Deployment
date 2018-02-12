IP Address: 52.14.21.109
URL: http://ec2-52-14-21-109.us-east-2.compute.amazonaws.com/

# Introduction
This web app runs on an Amazon Lightsail Ubuntu instance. With an apache server and a Flask application.

The following are the necessary steps that were taken to configure the web app:

# Get your server.
## 1. Start a new Ubuntu Linux server instance on Amazon Lightsail. 
## 2. Connect to instance using ssh
 ssh -i PrivateKey.pem ubuntu@52-14-21-109
 
## Secure your server
### 3. Updates all currently installed packages 
 sudo apt-get update
 sudo apt-get upgrade
 sudo apt-get install finger
 
## Give grader access
### 4. Create a new user named grader
 sudo adduser grader
 finger grader

### 5. Give grader the permission to sudo. First create the file and then add the second line to the file to give full access.
 sudo nano /etc/sudoers.d/grader
 grader ALL=(ALL:ALL) ALL
 
### 6. Create an SSH key pair for grader using the ssh-keygen tool
 ON LOCAL: ssh-keygen
 ON REMOTE:
 sudo mkdir /home/grader/.ssh
 sudo touch /home/grader/.ssh/authorized_keys
 COPY GENERATED PUB KEY TO: sudo nano authorized_keys
 
 Then change some permissions to only allow grader user full access to files:
 	sudo chmod 700 /home/grader/.ssh
 	sudo chmod 644 /home/grader/.ssh/authorized_keys
 	sudo chown -R grader:grader /home/grader/.ssh
 
 TRY LOGGIN IN WITH:  
 	ssh -i udacity_key grader@52-14-21-109
 
### 8. Add the Port 2200 as a Custom TCP Port in the Lightsail webpage and changed the SSH port from 22 to 2200 Firewall in the config file. 
ON THE LIGHTSAIL WEBSITE ADD PORT 2200 AS A CUSTOM TCP PORT. DELETE PORT SSH PORT 22.
ON THE SSHD_CONFIG FILE CHANGE PORT 22 TO 2200
sudo nano /etc/ssh/sshd_config
sudo service ssh restart

TRY LOGGIN IN WITH:  
ssh -i udacity_key grader@52-14-21-109 -p 2200

## Block Root login
sudo nano /etc/ssh/sshd_config
CHANGE PERMITROOTLOGIN TO "NO"

### 9. Set up all the other firewall ports
 sudo ufw default deny incoming
 sudo ufw default allow outgoing
 sudo ufw allow 2200/tcp
 sudo ufw allow 80/tcp
 sudo ufw allow 123/udp
 sudo ufw enable
 sudo ufw status

 TRY LOGGIN IN WITH:  
 	ssh -i udacity_key grader@52-14-21-109 -p 2200

## Prepare to deplay your project
### 7. Configure the local timezone
 (SET TO EUROPE -> LONDON) sudo dpkg-reconfigure tzdata
 sudo apt-get install ntp
 

## Install Apache and have it running with wsgi
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
sudo service apache2 start
PAGE SHOULD DISPLAY THE APACHE HTML. GET THE HOSTNAME WITH http://www.hcidata.info/host2ip.cgi

## Install GIT
sudo apt-get install git

## Set up the app within its own directory by downloading your own GIT repository
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
sudo git clone https://github.com/Ataboyata/Item_Catalog_Deployment FlaskApp

cd /var/www
sudo chown -R grader:grader FlaskApp

## Install Flask and Virtual Environment. Set up and activate the environment.
sudo apt-get install python-pip 
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate
sudo chmod -R 777 venv
pip install Flask requests oauth2client flask-httpauth sqlalchemy psycopg2 psycopg2-binary

## Configure the Application.
### Add the conf file so that Apache knows where it can take the available sites
sudo nano /etc/apache2/sites-available/FlaskApp.conf

ADDED THIS CONTENT:
<VirtualHost *:80>
                WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
                <Directory /var/www/FlaskApp/FlaskApp/>
                Order allow,deny
                Allow from all
                </Directory>
                Alias /static /var/www/FlaskApp/FlaskApp/static
                <Directory /var/www/FlaskApp/FlaskApp/static/>
                Order allow,deny
                Allow from all
                </Directory>
                        ErrorLog ${APACHE_LOG_DIR}/error.log
                        LogLevel warn
                        CustomLog ${APACHE_LOG_DIR}/access.log combined
 </VirtualHost>
THE FOLLOWING ENABLES THE FLASKAPP TO BE LOADED WITH APACHE
sudo a2ensite FlaskApp
THE FOLLOWING DISABLES THE DEFAULT APACHE PAGE
a2dissite 000-default

cd /var/www/FlaskApp/
sudo nano flaskapp.wsgi

ADDED THIS CONTENT:
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/FlaskApp/")
from FlaskApp import app as application
application.secret_key='super_secret_key'

## Configure PostgreSQL
sudo apt-get install libpq-dev python-dev
sudo apt-get install postgresql postgresql-contrib
sudo su - postgres
psql
CREATE USER catalog WITH PASSWORD 'grader';
ALTER USER catalog CREATEDB;
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit

### Finish creating the database and load it with premade items.
python /var/www/FlaskApp/FlaskApp/static/database_setup.py
python /var/www/FlaskApp/FlaskApp/static/lotsofitems.py
cd /var/www/FlaskApp/FlaskApp/static
python /var/www/FlaskApp/FlaskApp/static/__init__.py

sudo service apache2 restart

## Third Party Resources
* Google OAuth
* Apache
* Ubuntu
* Flask
* Amazon Lightsail