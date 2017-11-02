# Project: Linux Server Configuration
The final project of [Udacity](https://www.udacity.com): [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004)

This project requires students to configure an linux server to serve the content of a catalog website. The server is hosted on AWS(utilizing Amazon Lightsail).

* IP address: [54.238.214.236](http://54.238.214.236/)
* URL: [http://ec2-54-238-214-236.ap-northeast-1.compute.amazonaws.com/](http://ec2-54-238-214-236.ap-northeast-1.compute.amazonaws.com/)

# Secure the Server
#### 1. Log in the server, and configure the following settings
* Update the package list and install the packages
```
sudo apt-get update
sudo apt-get upgrade
```
* Edit the config file `/etc/ssh/sshd_config` and alter the following settings
```
Port 2200
PermitRootLogin no
PasswordAuthentication no
```
  Restart ssh
```
sudo service ssh restart
```
* Configure the firewall using the following commands
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/tcp
sudo ufw enable
```

#### 2. Configure Amazon Lightsail firewall
Go to the virtual machine's networking tab, and edit the firewall to allow port 2200 connection

# Create user for grading
1. Create a new user
```
sudo adduser grader
```
2. Give grader the permission to sudo by creating the `/etc/sudoers.d/grader` file

3. Use `ssh-keygen` to generate a key pair on local computer, and add the public key to the virtual machine's `/home/grader/.ssh/authorized_keys` file

#### Now the grader user can log in the machine via SSH remotely

# Configure the web server and database

#### 1. Configure the web server
* Configure the local timezone to UTC via:
```
sudo dpkg-reconfigure tzdata
```
* Install Apache
```
sudo apt-get install apache2
```

#### 2. Configure mod_wsgi  
* Create the file structure inside `/var/www` as
```
CatalogApp
  -- CatalogApp
     -- static
     -- templates
```
and create `__init__.py` file in the `/var/www/CatalogApp/CatalogApp` directory
* Install necessary packages
```
sudo apt-get install libapache2-mod-wsgi
sudo apt-get install python-dev python-pip
sudo apt-get install python-flask python-sqlalchemy
```
* Install Python packages
```
sudo pip install oauth2client httplib2 requests psycopg2
```
* Add the following file to configure the virtual host
```
sudo nano /etc/apache2/sites-available/CatalogApp.conf
```
Configure the file as below:
```
<VirtualHost *:80>
		ServerName http://54.238.214.236/
		ServerAdmin admin@54.238.214.236
		WSGIScriptAlias / /var/www/CatalogApp/catalogapp.wsgi
		<Directory /var/www/CatalogApp/CatalogApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/CatalogApp/CatalogApp/static
		<Directory /var/www/CatalogApp/CatalogApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Disable the default and enable the wsgi app
```
sudo a2dissite 000-default.conf
sudo a2ensite CatalogApp.conf
```
* Create the catalogapp.wsgi file
```
cd /var/www/CatalogApp
sudo nano catalogapp.wsgi
```

  Add the following content:
  ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/CatalogApp/")

  from CatalogApp import app as application
  application.secret_key = 'super_key'
  ```

* Restart Apache
```
sudo service apache2 restart
```

#### 3. Configure the database
* Install `postgresql`
```
sudo apt-get install postgresql
```
* Log in as default user
```
sudo su postgres
```
* Enter the shell with `psql` command and execute the following script to create the database and user for the app
```
CREATE USER catalog WITH PASSWORD 'cata-password';
CREATE DATABASE catalog;
GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
```
#### 4. Install Git, clone, and configure catalog project
* Install Git
```
sudo apt-get install git
```
* Clone and rename the project at `/var/www/CatalogApp`
```
sudo git clone https://github.com/alankashiwa/cafe-catalog-project.git
sudo mv ./cafe-catalog-project ./CatalogApp
```
* Change database engine setting in `database_setup.py`, `populate_db_items.py` and `project.py`
```
'postgresql://catalog:cata-password@localhost/catalog'
```
* Rename `project.py` to `__init__.py`
```
sudo mv project.py __init__.py
```
* Use absolute path for `client_secrets.json` and `fb_client_secrets.json` in `__init__.py`

# Reference
Part of the server configuration is based on the following article from [Digital Ocean: How to deploy a flask application on an ubuntu vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
