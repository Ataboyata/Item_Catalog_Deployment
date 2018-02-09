ec2-18-218-185-59.us-east-2.compute.amazonaws.com
# Get your server.
## 1. Start a new Ubuntu Linux server instance on Amazon Lightsail. 
## 2. Connect to instance using ssh
 ssh -i PrivateKey.pem ubuntu@13.59.157.20
 
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
 COPY GEN PUB KEY TO: sudo nano authorized_keys
 
 Then change some permissions:
 	sudo chmod 700 /home/grader/.ssh
 	sudo chmod 644 /home/grader/.ssh/authorized_keys
 	sudo chown -R grader:grader /home/grader/.ssh
 
 TRY LOGGIN IN WITH:  
 	ssh -i udacity_key grader@18.218.185.59
 
### 8. Add the Port 2200 as a Custom TCP Port in the Lightsail webpage and changed the SSH port from 22 to 2200 Firewall in the config file. 
ON THE LIGHTSAIL WEBSITE ADD PORT 2200 AS A CUSTOM TCP PORT. 
sudo nano /etc/ssh/sshd_config
CHANGE PORT 22 TO 2200

### 9. Set up all the other firewall ports
 sudo ufw default deny incoming
 sudo ufw default allow outgoing
 sudo ufw allow 2200/tcp
 sudo ufw allow 80/tcp
 sudo ufw allow 123/udp
 sudo ufw enable
 sudo ufw status

 TRY LOGGIN IN WITH:  
 	ssh -i udacity_key grader@18.218.185.59 p-2200

## Prepare to deplay your project
### 7. Configure the local timezone
 (SET TO EUROPE -> LONDON) sudo dpkg-reconfigure tzdata
 sudo apt-get install ntp
 
## Block Root login
sudo nano /etc/ssh/sshd_config
CHANGE PERMITROOTLOGIN TO "NO"

## Set up Apache
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
sudo a2enmod wsgi
sudo service apache2 start
PAGE SHOULD DISPLAY THE APACHE HTML. GET THE HOSTNAME WITH http://www.hcidata.info/host2ip.cgi

## Install GIT
sudo apt-get install git

## Set up folders
cd /var/www
sudo mkdir FlaskApp
cd FlaskApp
sudo git clone https://github.com/Ataboyata/Item_Catalog_Deployment FlaskApp

cd /var/www
sudo chown -R grader:grader FlaskApp

## Install Flask
sudo apt-get install python-pip 
sudo pip install virtualenv
sudo virtualenv venv
source venv/bin/activate 
sudo pip install Flask 
sudo pip install requests
sudo pip install oauth2client
sudo pip install flask-httpauth

## Configure the Application
sudo nano /etc/apache2/sites-available/FlaskApp.conf

ADDED THIS CONTENT:
<VirtualHost *:80>
                ServerName 18-218-151-163
                ServerAdmin admin@mywebsite.com
                WSGIScriptAlias / /var/www/FlaskApp/FlaskApp/flaskapp.wsgi
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
 
 
 
cd /var/www/FlaskApp/FlaskApp
sudo nano flaskapp.wsgi
ADDED THIS CONTENT:


