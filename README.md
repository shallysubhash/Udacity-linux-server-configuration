# Udacity-linux-server-configuration
This is the fifth project for "Full Stack Web Developer Nanodegree" on Udacity. In this project, a Linux virtual machine needs to be configured to support the Item Catalog website.
- IP address: 35.167.27.204
- Accessible SSH port: 2200
- Application URL : http://ec2-35-154-32-235.us-west-2.compute.amazonaws.com
# walkthrough
SSH into your server
- From the Account menu on Amazon Lightsail, click on SSH keys tab and download the Default Private Key.
- Move this private key file named LightsailDefaultPrivateKey-*.pem into the local folder ~/.ssh and rename it udacity_key.rsa.
- In your terminal, type: chmod 600 ~/.ssh/udacity_key.rsa
- Connect to the instance via the terminal using ssh -i .ssh/udacity_key.rsa ubuntu@35.154.32.235 -p
# Create a new user named grader
- Add a new user called grader: $ sudo adduser grader
- Create a new file in the sudoers directory with sudo nano /etc/sudoers.d/grader
- Add the following text grader ALL=(ALL:ALL) ALL
# Update all currently installed packages
- $ sudo apt-get update
- $ sudo apt-get upgrade
# Configure the local timezone to UTC
- $ sudo dpkg-reconfigure tzdata
- $ sudo apt-get install ntp
# Configure the key-based authentication
- Generate key pair locally using ssh-keygen
- $ su - grader
- $ mkdir .ssh
- $ touch .ssh/authorized_keys
- $ vim .ssh/authorized_keys
Copy the public key generated on your local machine to this file and change permissions
- $ chmod 700 .ssh
- $ chmod 644 .ssh/authorized_keys
Reload the SSH
# Change the SSH port from 22 to 2200
- $ sudo nano /etc/ssh/sshd_config. Find the Port line and edit it to 2200
- $ sudo service ssh restart
# Configure the Uncomplicated Firewall (UFW)
- $ sudo ufw allow 2200/tcp.
- $ sudo ufw allow 80/tcp.
- $ sudo ufw allow 123/udp.
- $ sudo ufw enable
# Install Apache, mod_wsgi
- Install Apache sudo apt-get install apache2
- Install mod_wsgi sudo apt-get install python-setuptools libapache2-mod-wsgi-py3
- Enable mod_wsgi using: sudo a2enmod wsgi
- Restart Apache sudo service apache2 restart
# Install Git
- $ sudo apt-get install git
# Clone the Catalog app from Github
- $ cd /var/www
- $ sudo mkdir catalog
- Move inside that newly created folder
- $ git clone https://github.com/shallysubhash/Item-Catalog catalog
- Create a catalog.wsgi file, and add
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
- Rename category.py to init.py mv category.py __init__.py
- In __init__.py, replace line below:
 app.run(host="0.0.0.0", port=8000, debug=True) to app.run()
 -In database.py, replace line 9:
 engine = create_engine("sqlite:///catalog.db") to engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')

# Install virtual environment
- install the virtual environment sudo pip3 install virtualenv
- Create a new virtual environment with sudo virtualenv venv2
- Activate the virutal environment source venv3/bin/activate
- Change permissions sudo chmod -R 777 venv3
- Install Flask: $ pip3 install Flask
- Install other project dependencies
sudo pip3 install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils

# Configure and enable a new virtual host
Create a virtual host conifg file: $ sudo nano /etc/apache2/sites-available/catalog.conf
-Copy paste the below lines
```
<VirtualHost *:80>
    ServerName 35.154.32.235
    ServerAlias ec2-35-154-32-235.us-west-2.compute.amazonaws.com
    ServerAdmin shallysubhash@35.154.32.235
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
- Enable the virtual host sudo a2ensite catalog

#Install and configure PostgreSQL
- sudo apt-get install postgresql
- Swith the user sudo su - postgres
- Open Postgres using psql
-Create user with a password and give them the ability to create database
CREATE ROLE catalog WITH LOGIN PASSWORD 'shally';
ALTER ROLE catalog CREATEDB;
- connect to the 'catalog' datbase \c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog;
\q
exit
- Create database using  $ python3 /var/www/catalog/catalog/datbase_setup.py
# Google Authentication
- Add site url 35.154.32.235.xip.io to Authorised domains field in OAuth consent screen
- Add site url with http http"//35.154.32.235.xip.io to Authorized JavaScript origins in Credentials Setting
- Add redirect link to Authorized redirect URIs in Credentials Setting
- Download client secret json file and rename to client_secrets.json
- Replace the client_secrets.json file with the updated version in server
- Edit flow_from_clientsecrets('client_secrets.json') to flow_from_clientsecrets('/var/www/category/category/client_secrets.json')

# Launch the Web Application at http://35.154.32.235.xip.io/
