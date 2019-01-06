# Linux-Server-Configuration
This project is required for Udacity's Full Stack Web Developer Nanodegree program.

## Project Overview
A baseline installation of a Linux server and preparing it to host our web applications, securing it from a number of attack vectors, installing and configuring a database server, and deploying our one of your existing web applications onto it.

### Link to Project: [ItemCatalog](http://54.93.245.143.xip.io/)

## IP & Hostname
1. IP: 54.93.245.143 
2. Hostname: ec2-54-93-245-143.eu-central-1.compute.amazonaws.com

## Amazon Lightsail Set Up
1. Go to the [Amazon Lightsail](https://lightsail.aws.amazon.com/) website
2. Click get started for free
3. Create your first instance
4. Select OS Only and Ubuntu 
5. Scroll down and name your instance whatever you'd like and click create.
6. It might take a minute or two for the instance to set up. Once running click on it and click "Account Page" at the bottom so you can download your private SSH key. 
7. Now click download to get your private key. The file type is .pem and will be used to SSH into the server.
8. The last thing we will need to do is configure the ports Amazon Lightsail will allow. By default the firewall is set to only allow connects from port 22 and port 80. We need to set up port 2200 and 123.
9. Click the networking tab 
10. From this tab click add another under "Firewall" and choose Custom for application, TCP for protocol, and the port number under Port Range. Then click save. 
## Linux Configuration
1. Download your instance Private Key from your profile.
2. in your local machine, cd to ( Users⁩ ▸ [your user] ▸ .ssh ).
3. move your private key file into this directory and name it 'udacity_key.pem'.
4. change permission using `chmod 600 ~/.ssh/udacity_key.pem`.
5. log into our Amazon Lightsail Server: `$ ssh -i ~/.ssh/udacity_key.pem ubuntu@[PUBLIC IP ADDRESS]`.
6. log in as root user: `sudo su -`.
7. Then type `sudo adduser grader` to create another user 'grader' 
8. `sudo nano /etc/sudoers.d/grader`.
9. add the following line `grader ALL=(ALL:ALL) ALL` to add grader as a user.
### update virtual machine packages
1. `sudo apt-get update`.
2. `sudo apt-get upgrade`.
3. `sudo apt-get dist-upgrade`.
4. `sudo apt-get install finger`.
### Configure the key-based authentication for grader user
1. in another terminal write this `ssh-keygen -f ~/.ssh/grader_key.rsa` it will ask for password that grader will use to access the machine.
2. `cat ~/.ssh/grader_key.rsa.pub` to find the public key. COPY IT.
3. Back to server terminal, `cd /home/grader`.
* create .ssh folder with `mkdir .ssh` for saving public keys.
* `touch .ssh/authorized_keys`
* `nano .ssh/authorized_keys` and paste the public key.
* change file permissions:
1. `sudo chmod 700 /home/grader/.ssh`.
2. `sudo chmod 644 /home/grader/.ssh/authorized_keys`.
3. change file owner `sudo chown -R grader:grader /home/grader/.ssh`.
4. restart server configuration `sudo service ssh restart`.
5. disconnect.
* Log into the server as grader: `ssh -i ~/.ssh/grader_key.rsa grader@54.93.245.143`
* change PasswordAuthentication to NO and change port to 22200 using `sudo nano /etc/ssh/sshd_config`.
* Restart ssh: '$ sudo service ssh restart'
* BEFORE disconnecting and accessing server with port 2200, we need to setup firewall or else we will lose our access to the server:
- sudo ufw allow 2200/tcp
- sudo ufw allow 80/tcp
- sudo ufw allow 123/udp
- sudo ufw enable
* Diconnect and log in using `ssh -i ~/.ssh/grader_key.rsa grader@54.93.245.143 -p 2200`
## Application Deployment
Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
1. Start by installing the required software
```
 $ sudo apt-get install apache2
 $ sudo apt-get install libapache2-mod-wsgi python-dev
 $ sudo apt-get install git
 ```
 2. Enable mod_wsgi with the command '$ sudo a2enmod wsgi' and restart Apache using '$ sudo service apache2 restart'.
 3. Set up the folder structure :
 ```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
$ git clone [Github Link] catalog
```
4. create .wsgi file `sudo nano catalog.wsgi` in the same directory.






