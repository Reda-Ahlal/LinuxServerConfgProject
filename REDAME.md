# Overview
This is the third and last project for the Udacity Full Stack Nanodegree. This project involves taking a baseline installation of Linux on a virtual machine and preparing it to host web applications. This includes installing updates, securing the server from attacks, and installing / configuring web and database servers.

# Server Info
* Public IP address: 18.184.220.251
* SSH Port: 2200
* URL to hosted web application: http://18.184.220.251

# Getting Started

### 1. Get a server
* Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/) using an Amazon Web Services account.
* Create an instance
* Choose an instance image: Ubuntu (OS only)
* Choose your instance plan (lowest 3.5$/Month)
* Give your instance a name
* Wait for startup
* Once the instance has started up, follow the instructions provided to SSH into your server.

### 2. SSH into the server
* From the Account menu on Amazon Lightsail, click on SSH keys tab and download the Default Private Key.
* Move this private key file named LightsailDefaultPrivateKey-.pem into the local folder ~/.ssh and rename it lightsail_key.rsa.
* In your terminal, type: chmod 600 ~/.ssh/lightsail_key.rsa.
To connect to the instance via the terminal: ssh -i ~/.ssh/lightsail_key.rsa ubuntu@18.184.220.251.

### 3. Secure the server
Update and upgrade installed packages
* sudo apt-get update
* sudo apt-get upgrade

Change the SSH port from 22 to 2200
* Edit the /etc/ssh/sshd_config file: sudo nano /etc/ssh/sshd_config.
* Change the port number from 22 to 2200.
* Save and exit using CTRL+X and confirm with Y.
* Restart SSH: sudo service ssh restart.

Configure the Uncomplicated Firewall (UFW)
Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

* sudo ufw status                  
* sudo ufw default deny incoming   
* sudo ufw default allow outgoing  
* sudo ufw allow 2200/tcp          
* sudo ufw allow www               
* sudo ufw allow 123/udp          
* sudo ufw deny 22                 
* Turn UFW on: sudo ufw enable.

Exit the SSH connection: type exit.

Click on the Manage option of the Amazon Lightsail Instance, then the Networking tab, and then change the firewall configuration to allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.

From your local terminal, run: ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@18.184.220.251.

### 4. Give grader access

##### Create a new user account named grader

 * add user: sudo adduser grader.
 * Enter a password (twice) and fill out information for this new user.

##### Give grader the permission to sudo

* Edits the sudoers file: sudo visudo.
* Search for the line that looks like this:
```
root    ALL=(ALL:ALL) ALL
```
* Below this line, add a new line to give sudo privileges to grader user.
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
```
* Save and exit using CTRL+X and confirm with Y.

##### Create an SSH key pair for grader using the ssh-keygen tool

* On the local machine: Run ssh-keygen
* Enter file in which to save the key (I gave the name grader_key) in the local directory ~/.ssh
Enter in a passphrase twice. Two files will be generated ( ~/.ssh/grader_key and ~/.ssh/grader_key.pub)
* Run cat ~/.ssh/grader_key.pub and copy the contents of the file
* Log in to the grader's virtual machine
* Create a new directory called ~/.ssh (mkdir .ssh)
Run sudo nano ~/.ssh/authorized_keys and paste the content into this file, save and exit
* Give the permissions: chmod 700 .ssh and chmod 644 .ssh/authorized_keys
* Check in /etc/ssh/sshd_config file if PasswordAuthentication is set to no
* Restart SSH: sudo service ssh restart
* On the local machine, run: ssh -i ~/.ssh/grader_key -p 2200 grader@18.184.220.251.

### 5. deploy the project

##### Install and configure Apache to serve a Python mod_wsgi application
* Configure the local timezone to UTC: sudo dpkg-reconfigure tzdata.
* install Apache: sudo apt-get install apache2.
* install the Python 2 mod_wsgi package:
sudo apt-get install libapache2-mod-wsgi.
* Enable mod_wsgi using: sudo a2enmod wsgi

##### Install and configure PostgreSQL
* install PostgreSQL: sudo apt-get install postgresql.
* Switch to the postgres user: sudo su - postgres.
* Open PostgreSQL interactive terminal with psql.
* Create the catalog user with a password and give them the ability to create databases:
```
postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
postgres=# ALTER ROLE catalog CREATEDB;
```
* Exit psql: \q.
* Switch back to the grader user: exit.
* Create a new Linux user called catalog: sudo adduser catalog. Enter password and fill out information.
* Give to catalog user the permission to sudo. Run: sudo visudo.
* add a new line to give sudo privileges to catalog user.
```
root    ALL=(ALL:ALL) ALL
grader  ALL=(ALL:ALL) ALL
catalog  ALL=(ALL:ALL) ALL
```
* Save and exit using CTRL+X and confirm with Y.
* While logged in as catalog, create a database: createdb catalog.
* Exit psql: \q.
* Switch back to the grader user: exit.

##### Install git
* While logged in as grader, install git: sudo apt-get install git.

##### Clone and setup the Item Catalog project from the GitHub repository

* While logged in as grader, create /var/www/catalog/ directory.
* Change to that directory and clone the catalog project:
sudo git clone https://github.com/Reda-Ahlal/UdacityCatalogApp.git catalog.
* From the /var/www directory, change the ownership of the catalog directory to grader using: sudo chown -R grader:grader catalog/.
* Change to the /var/www/catalog/catalog directory.
* Rename the application.py file to app.py using: mv application.py app.py.
* In app.py file, replace the line:
app.run(host='0.0.0.0', port=5000)
by
app.run()
and CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']
by
CLIENT_ID = json.loads(
    open('var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']
* In add_categories_to_database.py, database_setup.py and app.py files replace: engine = create_engine('sqlite:///moviescatalog.db') by
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

##### Install the virtual environment

* install pip: sudo apt-get install python-pip.
* Install the virtual environment: sudo pip install virtualenv
* Change to the /var/www/catalog/catalog/ directory.
* Create the virtual environment: sudo virtualenv venv.
* Change the ownership to grader with: sudo chown -R grader:grader venv/.
* Activate the new environment: . venv/bin/activate.
* Install httplib2 : pip install httplib2
* Install requests: pip install requests
* Install sqlalchemy: pip install sqlalchemy
* Install flask: pip install flask
* Install libpq-dev: sudo apt-get install libpq-dev
* Install psycopg2: pip install psycopg2

##### Set up and enable a virtual host

* Create /etc/apache2/sites-available/catalog.conf and add the following lines to configure the virtual host:
```
<VirtualHost *:80>
    ServerName 18.184.220.251
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
* Enable virtual host: sudo a2ensite catalog.
* Reload Apache: sudo service apache2 reload.

##### Set up the Flask application

* Create /var/www/catalog/catalog.wsgi file add the following lines:

```
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from app import app as application
application.secret_key = "secret_key"
```
* Restart Apache: sudo service apache2 restart.

##### Disable the default Apache site

* Disable the default Apache site: sudo a2dissite 000-default.conf.
* Reload Apache: sudo service apache2 reload.

##### Launch the Web Application

* Restart Apache again: sudo service apache2 restart.
* Open your browser to http://18.184.220.251
