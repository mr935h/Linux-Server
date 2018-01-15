# Udacity - Linux Server Configuration Project 5

## Description
    This project was create from an instance on Amazon Lightsail web
    server platform to configure a Linux server and display my catalog
    application.

## Project information
    IP address: 18.217.132.149
    SSH port: 2200
    Hosted web application URL: https://lightsail.aws.amazon.com/ls/webapp/us-east-2/instances/Linux-Server-Project-5/connect

## Installed Software and configuration changes
  * apache2
  * postgresql
  * python2.7
  * python3
  * git
  * finger
  * ntp

## Steps to configure server

### Logging into server

1. Within a terminal run command below to ssh into server as *root*:`$ ssh root@[ip_of_server]`

### Create a new user with sudo permissions

1. Create a new user: `$ sudo adduser [username]`
2. Create a new file under suoders directory: `$ sudo nano /etc/sudoers.d/[username]`
3. The new file should contain the following:
    **[username]** ALL=(ALL:ALL) ALL
    * Note: to save this file use **control + x**

### Configure server for key-based authentication

1. Within a new terminal generate an encryption key **on your local machine** with: `$ ssh-keygen -f ~/.ssh/[file_name].rsa`
2. As *root* create a new file on the server to save the public key: `$ touch /home/[username]/.ssh/authorized_keys`
3. Add the public key from your local machine found at *~/.ssh/[filename].rsa.pub* to the server file at */home/[username]/.ssh/authorized_keys*.
4. Use the *change mode (chmod)* command to update permissions.
    1. `$ sudo chmod 700 /home/[username]/.ssh`
    2. `$ sudo chmod 644 /home/[username]/.ssh/authorized_keys`
5. Use the *change ownership (chown)* command to update ownership of the .ssh directory: `$ sudo chown -R [username]:[username] /home/[username]/.ssh`
4. Key-based authentication can now be used to ssh into server with the following command: `$ ssh [username]@[ip_of_server] -i ~/.ssh/[filename].rsa`

### Remove password authentication to only allow key-based authentication

1. Update *sshd_config* file: `$ sudo nano /etc/ssh/sshd_config`
    * Update this line to *no*: PasswordAuthentication *no*
2. Restart ssh: `$ sudo service ssh restart`

### Change the ssh port from 22 to 2200

1. Change sshd_config file to look for port 2200 instead of 22: `$ sudo nano /etc/ssh/sshd_config`
    * Update *Port 22* to *Port 2200*
2. Restart ssh: `$ sudo service ssh restart`
3. The following command can now be used to ssh into server: `$ ssh [username]@[ip_of_server] -p 2200 -i ~/.ssh/[file_name].rsa`

### Disable ssh login for *root* user

1. Change sshd_config file to not allow root user login: `$ sudo nano /etc/ssh/sshd_config`
    * Update *PermitRootLogin yes* to *PermitRootLogin no*
2. Restart ssh: `$ sudo service ssh restart`

### Update currently installed server software

1. Update packages: `$ sudo apt-get update`
2. Upgrade packages: `$ sudo apt-get upgrade`

### Install Finger - Utility Software to Check Users' Status

Install Finger: `$ apt-get install finger`

### Make the server timezone UTC

Reconfigure server timezone: `$ sudo dpkg-reconfigure tzdata`

### Configure the Uncomplicated Firewall (UFW)

Project required that firewall allow SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. `$ sudo ufw allow 2200/tcp`
2. `$ sudo ufw allow 80/tcp`
3. `$ sudo ufw allow 123/udp`
4. `$ sudo ufw enable`

Show existing configuration with the following command: `$ sudo ufw status numbered`

Status: active

```
     To                         Action      From
     --                         ------      ----
[ 1] 2200/tcp                   ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 123/udp                    ALLOW IN    Anywhere
[ 4] 2200/tcp (v6)              ALLOW IN    Anywhere (v6)
[ 5] 80/tcp (v6)                ALLOW IN    Anywhere (v6)
[ 6] 123/udp (v6)               ALLOW IN    Anywhere (v6)
```

Rules can be deleted by using the following command: `$ sudo ufw delete` + line corresponding to UFW status above ^^^

### Configure server for Automatic Security Updates

1. Install *unattended-upgrades*: `$ sudo apt-get install unattended-upgrades`
2. Enable *unattended-upgrades*: `$ sudo dpkg-reconfigure --priority=low unattended-upgrades`

### Install pip and Python packages

1. Install *pip* (Python package installer): `$ sudo apt-get install python-pip`
2. Install other Python packages needed for this Flask application: `$ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`

### Install Apache and mod_wsgi

1. Install *apache2*: `$ sudo apt-get install apache2`
2. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
    * Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications.
3. Enable *mod_wsgi*: `$ sudo a2enmod wsgi`
4. Restart *apache2*: `$ sudo service apache2 start`

### Install Git

1. Install git: `$ sudo apt-get install git`
2. Configure your username: `$ git config --global user.name <username>`
3. Configure your email: `$ git config --global user.email <email>`

### Clone a Flask Repo from Github to your server

1. Move to where you want to make the new directory: `$ cd /var/www`
2. Make new directory: `$ sudo mkdir [dir_name]`
2. Change owner for the *[dir_name]* folder: `$ sudo chown -R [username]:[username] [dir_name]`
3. While inside the new directory clone a repository from Github: `$ git clone [url_of_repo] [2nd_dir_name]`

Your directory structure should now look like this:
```
    |----[dir_name]
    |---------[2nd_dir_name]
    |--------------static
    |--------------templates
    |--------------__init__.py
```

### Create the .wsgi File
1. Move into [dir_name] directory: `$ cd /var/www/[dir_name]`
2. Make a .wsgi file.: `$ sudo nano [file_name].wsgi`

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/[dir_name]/")

from 2nd_dir_name import app as application
application.secret_key = 'Add your secret key'
```

### Get server ready to host Flask app

1. Use *pip* to install *virtualenv*: `$ sudo pip install virtualenv`
2. Within the [dir_name] directory create virtual environment: `$ sudo virtualenv Virtenv`
3. Activate the virtual environment: `$ source Virtenv/bin/activate`
4. Change permissions *Virtenv* folder: `$ sudo chmod -R 777 Virtenv`
5. Install Flask: `$ pip install Flask`

### Configure and Enable a New Virtual Host

1. Create a virtual environment configuration file: `$ sudo nano /etc/apache2/sites-available/[file_name].conf`

```
<VirtualHost *:80>
        ServerName [ip_of_server] or [url]
        ServerAdmin [email for administration issues]
        WSGIScriptAlias / /var/www/[dir_name]/[file_name].wsgi
        <Directory /var/www/[dir_name]/[2nd_dir_name]/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/[dir_name]/[2nd_dir_name]/static
        <Directory /var/www/[dir_name]/[2nd_dir_name]/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
2. Enable the new virtual host: `$ sudo a2ensite [file_name]`

Sources: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

https://devops.profitbricks.com/tutorials/deploy-a-flask-application-on-ubuntu-1404/

### Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`

    * postgres user is created when installing PostgreSQL.

3. Get into PostgreSQL as postgres user: `$ sudo su - postgres`
4. Connect to the database: `$ psql`
5. Create a new user and password: `# CREATE USER [new_user] WITH PASSWORD [password];`
5. Allow [new_user] CREATEDB capability: `# ALTER USER catalog CREATEDB;`
6. Create [new_database] database: `# CREATE DATABASE [new_database] WITH OWNER [new_user];`
7. Connect to the database: `# \c [new_database`
8. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`
9. Update permissions for [new_user]: `# GRANT ALL ON SCHEMA public TO [new_user];`
10. Getting out of database and logging out of PostgreSQL
    1. `# \q`
    2. `$ exit`
11. Double check the *pg_hba.conf* file to make sure it does not allow remote connections: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf`

Make sure the file looks like the below:

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Source: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

### Install Glances - System Monitor In Ubuntu

1. Make sure all packages are up-to-date: `$ sudo apt-get update`
2. Install glances: `$ sudo apt-get install glances`
3. Start glances: `$ glances`
4. Glances help: `$ glances -h`

Source: http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/

### Restart Apache to launch the app

1. Restart Apache: `$ sudo service apache2 restart`

## Changes to my Existing Catalog app

1. Google OAuth needed to allow the server ip to work
2. Changed Flask application python file from *project.py* to *'__init__.py'*
3. Set up new PostgreSQL database with SQLAlchemy by updating and running *setup.py* by following the below steps:
    1. Add *import psycopg2*
    2. Update *engine* for postgresql
        ```engine = create_engine('postgresql://catalog:grader@localhost/catalog')```
    3. Within Catalog directory run python setup file: `$ python setup.py`
4. Added *settings.py* file to locate *client_secrets.json* file

    ```import os
    # __file__ refers to the file settings.py
    APP_ROOT = os.path.dirname(os.path.abspath(__file__))   # refers to application_top
    ```

5. Updates to *'__init__.py'*
    1. Connection to database
        ```engine = create_engine('postgresql://catalog:[password]@localhost/catalog')```
    2. OAuth *client_secrets.json* file could not be found added APP_ROOT from settings file

        ```CLIENT_ID = json.loads(open(APP_ROOT + '/client_secrets.json', 'r').read())['web']['client_id']```

        ```oauth_flow = flow_from_clientsecrets(APP_ROOT + '/client_secrets.json', scope='')```

## 3rd Party
I reference iliketomatoes linix_server_configuration GitHub repo for helpful information on this project.
