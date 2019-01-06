# Linux-Server-Configuration
This project is required for Udacity's Full Stack Web Developer Nanodegree program.

## Project Overview
A baseline installation of a Linux server and preparing it to host a web applications, securing it from a number of attack vectors, installing and configuring a database server, and deploying one of your existing web applications onto it.

### Link to Project: [ItemCatalog](http://54.93.245.143.xip.io/)

## IP & Hostname
1. IP: 54.93.245.143 
2. Hostname: ec2-54-93-245-143.eu-central-1.compute.amazonaws.com

## Amazon Lightsail Set Up
1. Go to the [Amazon Lightsail](https://lightsail.aws.amazon.com/) website.
2. Click get started for free.
3. Create your instance *(A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter)*.
4. Select "OS Only" and "Ubuntu".
5. Name your instance.You can use any name you like, as long as it doesn't have spaces or unusual characters in it. Then click "create".
6. It may take a few minutes for your instance to start up. Once running click on it and click "Account Page" at the bottom so you can   download your "private SSH key". 
7. Now click download to get your private key. The file type is .pem and will be used to SSH into the server.
8. Click the 'Networking' tab and find the 'Add another' under "Firewall". choose Custom for application, TCP for protocol, and add ports 123 and 2200.
## Linux Configuration
1. Download your instance Private Key from your profile.
2. In your local machine, cd to ( Users⁩ ▸ [your user] ▸ .ssh ).
3. Move your private key file into this directory and name it 'udacity_key.pem'.
4. To make our key secure use `chmod 600 ~/.ssh/udacity_key.pem`.
5. log into our Amazon Lightsail Server as the user ubuntu with our key: `$ ssh -i ~/.ssh/udacity_key.pem ubuntu@[PUBLIC IP ADDRESS]`.
6. log in as root user: `sudo su -`.
7. Type `sudo adduser grader` to create another user 'grader'.
8. Create a file to give the user grader superuser privileges: `sudo nano /etc/sudoers.d/grader`.
9. Add the following line `grader ALL=(ALL:ALL) ALL` to add grader as a user.
### update virtual machine packages
1. `sudo apt-get update`.
2. `sudo apt-get upgrade`.
3. `sudo apt-get dist-upgrade`.
4. `sudo apt-get install finger`.
### Configure the key-based authentication for "grader" user
1. From a new terminal run the command `ssh-keygen -f ~/.ssh/grader_key.rsa`.
2. Run `cat ~/.ssh/grader_key.rsa.pub` to find the public key. Copy it.
3. Back to server terminal, `cd /home/grader`.
   * create .ssh folder with `mkdir .ssh` for saving public keys.
   * Create a file to store the public key with the command `touch .ssh/authorized_keys`
   * Edit that file using `nano .ssh/authorized_keys` and paste the public key.
   * Change file permissions using:
     1. `sudo chmod 700 /home/grader/.ssh`.
     2. `sudo chmod 644 /home/grader/.ssh/authorized_keys`.
     3. change file owner `sudo chown -R grader:grader /home/grader/.ssh`.
     4. restart server configuration `sudo service ssh restart`.
     5. disconnect.
* Log into the server as grader: `ssh -i ~/.ssh/grader_key.rsa grader@54.93.245.143`
* Change PasswordAuthentication to NO and change port to 2200 using `sudo nano /etc/ssh/sshd_config`.
* Restart ssh: `$ sudo service ssh restart`.
* From here we need to configure the firewall using these commands:
  ```
  $ sudo ufw allow 2200/tcp
  $ sudo ufw allow 80/tcp
  $ sudo ufw allow 123/udp
  $ sudo ufw enable
  ```
* Diconnect and log in using `ssh -i ~/.ssh/grader_key.rsa grader@54.93.245.143 -p 2200`
## Application Deployment
Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
1. Install required packages
```
 $ sudo apt-get install apache2
 $ sudo apt-get install libapache2-mod-wsgi python-dev
 $ sudo apt-get install git
 ```
 2. Enable mod_wsgi with the command `$ sudo a2enmod wsgi` and restart Apache using `$ sudo service apache2 restart`.
 3. Set up the folder structure:
 ```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
$ git clone [Github Link] catalog
```
4. Create the .wsgi file in the same directory by `$ sudo nano catalog.wsgi` and make sure your secret key matches with your project secret key.
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
5. Rename your item_catalog.py or whatever you called it in your catalog application folder to `__init__.py` by: 
`$ mv item_catalog.py __init__.py`
6. Install the virtual machine, make sure you are in `/var/www/catalog`:
```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```
7. Install All packages required for your application:
```
$ sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install sqlalchemy
$ pip install psycopg2-binary
$ pip install requests
$ pip install oauth2client
```
8. Nano to `__init__.py` and change `client_secrets.json` path to `/var/www/catalog/catalog/client_secrets.json`
9. Now we need to configure and enable the virtual host with `sudo nano /etc/apache2/sites-available/catalog.conf`
10. Paste the following code and save:

```
<VirtualHost *:80>
    ServerName [YOUR PUBLIC IP ADDRESS]
    ServerAlias [YOUR AMAZON LIGHTSAIL HOST NAME]
    ServerAdmin admin@[YOUR PUBLIC IP ADDRESS]
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
If you need help finding your servers hostname go [here](https://whatismyipaddress.com/ip-hostname) and paste the IP address.
11. Enable to virtual host:
`$ sudo a2ensite catalog.conf`.
and DISABLE the default host:
`$ a2dissite 000-default.conf`.
to enable your site and loading with the hostname.
### setup the database
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres
$ psql
```
### Create a database user and password
```
postgres=# CREATE USER catalog WITH PASSWORD [your password];
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```
Now go and change `database_setup` and `seeder` and `__init__` files to have instead of `sqlite:///databasename.db` to `postgresql://catalog:[your password]@localhost/catalog`.
### Setup Google oAuth:
1. Add authorized domains `xip.io` at google console.
2. Add authorized Javascript origins: `http://54.93.245.143.xip.io`
3. Add authorised redirect URIs: `http://54.93.245.143.xip.io/gconnect` and `http://54.93.245.143.xip.io/login`
4. Download the updated JSON file, copy its content and paste it in server copy : `sudo nano client_secrets.json`.

</br>Finally, Restart your apache server `$ sudo service apache2 restart` and open up your ip address in the browser with extension `xip.io`.

### References:
* https://github.com/mulligan121/Udacity-Linux-Configuration
* https://github.com/iMishaDev/linux-server-configuration






