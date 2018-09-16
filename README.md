# Project: Linux Server Configuration
This project deploys a web application for an item catalog onto a Linux server. The server is secured from various attack vectors and configures a database server.

This project is an assignment for the [Udacity Full Stack Web Developer Nanodegree course](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). 

## Server Information
- IP Address: 54.245.214.174
- Server URL: http://ec2-54-245-214-174.us-west-2.compute.amazonaws.com
- SSH port: 2200

## Installed Software
- apache2
- flask
- git
- httplib2
- libapache2-mod-wsgi
- oauth2client
- postgresql
- python-dev
- python-pip
- python-psycopg2
- request 
- Sqlalchemy
- virtualenv

## 1) Creating a New User
- On your local machine, generate an encryption key for user grader: ```ssh-keygen ~/.ssh```
- rename the file to grader.pem: ```mv grader grader.pem```
- Copy contents of grader.pub: ```cat ~./ssh/grader.pub```
- create the user: ```sudo adduser grader```
- make a new file: ```sudo nano /etc/sudoers.d/grader```
- paste this text into file and save: ```grader ALL=(ALL:ALL) ALL```
- access grader: ```sudo su - grader```
- paste contents from grader.pub into authorized_keys file: ```sudo nano ~/.ssh/authorized_keys```
- give access from root to grader: ```chown grader:grader /home/grader/.ssh/```
- Now, you should be able to access grader remotely with: ```ssh grader@54.245.214.174 -p 2200 -i ~/.ssh/grader.pem```

## 2) Enable key-based authentication
- ```sudo nano /etc/ssh/sshd_config``` and change ```passwordAuthentication``` to "no"

## 3) Disable root user remote login 
- ```sudo nano /etc/ssh/sshd_config``` and change ```permitRootLogin``` to "no"

## 4) Update SSH port from 22 to 2200
- ```sudo nano /etc/ssh/sshd_config``` and change ```Port``` to "2200"

## 5) Configure firewall to allow connections for SSH port 2200, HTTP port 80, and NTP port 123
- ```sudo ufw default deny incoming```
- ```sudo ufw default allow outgoing```
- ```sudo ufw allow 2200/tcp```
- ```sudo ufw allow www```
- ```sudo ufw allow 123/udp```
- ```sudo ufw deny 22```
- ```sudo ufw enable```
- ```sudo ufw status```
### Amazon Lightsail Network Configuration
- Go to your Amazon Lightsail account on the browser
- View Networking tab and make sure firewall settings have the following:
HTTP TCP 80
CUSTOM UDP 123
CUSTOM TCP 2200

## 6) Database setup
- Install PostgreSQL: ```sudo apt-get install postgresql```
- Switch to user postgres: ```sudo su - postgres```
- Connect to DB: ```psql```
- Create a user catalog: ```CREATE USER catalog WITH PASSWORD 'catalog';```
- Create to a database for catalog with owner as catalog: ```CREATE DATABASE catalog WITH OWNER catalog;```
- Connect to catalog database: ```\c catalog```
- Revoke all rights: ```REVOKE ALL ON SCHEMA public FROM public;```
- Grant permission to only catalog user: ```GRANT ALL ON SCHEMA public TO catalog;```
- ```\q``` and ```exit``` to get out of PostgreSQL

# 7) App Setup and Git
- Install Git: ```sudo apt-get install git```
- Access www directory: ```cd /var/www```
- Create a new directory called catalog: ```sudo mkdir catalog```
- Change owner for catalog directory: ```sudo chown -R grader:grader catalog```
- Access catalog directory: ```cd catalog```
- Clone catalog repository: ```git clone https://github.com/huynhkev7/catalog.git```
- Go into catalog directory containing files from branch: ```cd catalog```
- Rename python file: ```mv app.py __init__.py```
- Edit file ```sudo nano __init__.py``` and replace database connection to ```postgresql://catalog:catalog@localhost/catalog```
- Edit file ```sudo nano setup_database.py``` and replace database connection to ```postgresql://catalog:catalog@localhost/catalog```
- Edit file ```sudo nano populate_database.py``` and replace database connection to ```postgresql://catalog:catalog@localhost/catalog```
- Edit file ```sudo nano __init__.py``` and update path for ```client_secrets.json``` to ```/var/www/catalog/catalog/client_secrets.json```
# 8) Configuring virtual hosting
- Install apache2: ```sudo apt-get install apache2```
- Install wsgi: ```sudo apt-get install libapache2-mod-wsgi```
- Enable wsgi: ```sudo a2enmod wsgi```
- ```sudo service apache2 start```
- Navigate to catalog folder: ```cd /var/www/catalog```
- Create a catalog.wsgi file and paste content: 
    ```
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")
    from catalog import app as application
    ```
- ```sudo nano /etc/hosts``` and add ```ec2-54-245-214-174.us-west-2.compute.amazonaws.com```
- Create a virtual host config file ```sudo nano /etc/apache2/sites-available/catalog.conf``` and paste content:
```
<VirtualHost *:80>
        ServerName 54.245.214.174
        ServerAlias ec2-54-245-214-174.us-west-2.compute.amazonaws.com
        ServerAdmin huynhdikev@gmail.com
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-pac$
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
- Enable virtual host: ```sudo a2ensite catalog```

# 9) Configure Virtual Environment 
- Go to project catalog directory: ```cd /var/www/catalog```
- Install pip: ```sudo apt-get install python-pip```
- Install virtual environment: ```sudo pip install virtualenv```
- Create virtual environment: ```sudo virtualenv venv```
- Activate virtual environment: ```source venv/bin/activate```
- Install dependencies:
    ```
    sudo pip install Flask
    sudo pip install sqlalchemy
    sudo pip install psycopg2
    sudo pip install oauth2client
    sudo pip install httplib2
    sudo pip install requests
    ```

# 10) Update OAuth for Google
- Go to: https://console.cloud.google.com/apis
- Select Credentials
- Select App Catalog under OAuth 2.0 client IDs
- Add the following origins:
```http://ec2-54-245-214-174.us-west-2.compute.amazonaws.com```
```http://54.245.214.174```
- Add the following redirect URIs
```http://ec2-54-245-214-174.us-west-2.compute.amazonaws.com/login```
```http://ec2-54-245-214-174.us-west-2.compute.amazonaws.com/gconnect```

# 11) Restart Server and View Webpage
- restart server: ```sudo service apache2 restart```
- visit http://ec2-54-245-214-174.us-west-2.compute.amazonaws.com to see the web app in action!

## 3rd Part Resources
- https://aws.amazon.com/lightsail/
- https://github.com/iliketomatoes/linux_server_configuration

## Authors
Created by Kevin Huynh