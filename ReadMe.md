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
 COPY GEN KEY TO: sudo nano authorized_keys
 
 TRY LOGGIN IN WITH:  ssh -i udacity_key.rsa grader@18.218.185.59 -p 2200
 
 Then change some permissions:
 	sudo chmod 700 /home/grader/.ssh
 	sudo chmod 644 /home/grader/.ssh/authorized_keys
 	sudo chown -R grader:grader /home/grader/.ssh

## Prepare to deplay your project
### 7. Configure the local timezone
 (SET TO EUROPE -> LONDON) sudo dpkg-reconfigure tzdata
 sudo apt-get install ntp

### 8. Add the Port 2200 as a Custom TCP Port in the Lightsail webpage and changed the SSH port from 22 to 2200 Firewall in the config file. 
 sudo nano /etc/ssh/sshd_config
 sudo service ssh restart

### 9. Set up all the other firewall ports
 sudo ufw default deny incoming
 sudo ufw default allow outgoing
 sudo ufw allow 2200/tcp
 sudo ufw allow 80/tcp
 sudo ufw allow 123/udp
 sudo ufw enable
 sudo ufw status


