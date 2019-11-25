# FSND-Linux-server-configuration
This is the last project toward Udacity Full Stack Web Developer Nanodegree.
 my project was a [catalog for tvshows](https://github.com/moussaab-moulim/tvshows-catalog) .
  the project will be hosted on an Ubuntu Linux server on an Amazon Lightsail instance.
   A series of instructions will be presented below. You can visit [here](http://35.181.55.110/) for the website deployed. 

Above link will be unavailable after the amazon lightsail trial 

* Public IP address: 35.181.55.110
* SSH port: 2200
## Start a new Ubuntu Linux Server instance on Amazon Lightsail
1. Create an AWS account
2. Click **Create instance** button on the home page
3. Select **Linux/Unix** platform
4. Select **OS Only** and **Ubuntu** as blueprint
5. Select an instance plan
6. Name your instance
7. Click **Create** button

## SSH into your Server
1. Download private key from the **SSH keys** section in the **Account** section on Amazon Lightsail. The file name should be some thing like _LightsailDefaultPrivateKeyxxxxx.pem_
2. Create a new file named **lightsail_key.rsa** under ~/.ssh folder on your local machine
3. Copy and paste content from downloaded private key file to **lightsail_key.rsa**
4. Set file permission as owner only : `$ chmod 600 ~/.ssh/lightsail_key.rsa`
5. SSH into the instance: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@35.181.55.110`

## Update all currently installed packages
1. Run `sudo apt-get update` to update packages
2. Run `sudo apt-get upgrade` to install newest versions of packages
3. Set for future updates: `sudo apt-get dist-upgrade`

##  Change the SSH port from 22 to 2200
1. Run `$ sudo nano /etc/ssh/sshd_config` to open up the configuration file
2. Change the port number from **22** to **2200** in this file
3. Save and exit the file
4. Restart SSH: `$ sudo service ssh restart`

## Configure the firewall
1. Check firewall status: `$ sudo ufw status`
2. Set default firewall to deny all incomings: `$ sudo ufw default deny incoming`
3. Set default firewall to allow all outgoings: `$ sudo ufw default allow outgoing`
4. Allow incoming TCP packets on port 2200 to allow SSH: `$ sudo ufw allow 2200/tcp`
5. Allow incoming TCP packets on port 80 to allow www: `$ sudo ufw allow www`
6. Allow incoming UDP packets on port 123 to allow NTP: `$ sudo ufw allow 123/udp`
7. Close port 22: `$ sudo ufw deny 22`
8. Enable firewall: `$ sudo ufw enable`
9. Check out current firewall status: `$ sudo ufw status`
10. Update the firewall configuration on Amazon Lightsail website under **Networking**. add **port 80, 123, 2200**
11. Open up a new terminal and you can now ssh in via the new port 2200: `$ ssh -i ~/.ssh/lightsail_key.rsa ubuntu@YOUR_SERVER_PUBLIC_IP -p 2200`

## Create a new user account **grader** and give **grader** sudo access 
1. Create a new user account **grader**:`$ sudo adduser grader`
2. `usermod -aG sudo ugrader`


## Set SSH login using keys
1. Create an SSH key pair for **grader** using the `ssh-keygen` tool on your local machine. Save it in `~/.ssh` path
2. Deploy public key on development environment
    * On your local machine, read the generated public key
     `cat ~/.ssh/yourfile.pub`
    * On your virtual machine
   ```$ su -grader
      $ mkdir .ssh
      $ touch .ssh/authorized_keys
      $ nano .ssh/authorized_keys
      ```
    * Copy the public key from `~/.ssh/FILE-NAME.pub` to this _authorized_keys_ file on the remote and save
3. Run `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys` on remote server to change file permission
4. Restart SSH: `$ sudo service ssh restart`
5. Now you are able to login in as grader: `$ ssh -i ~/.ssh/grader_key -p 2200 grader@YOUR_SERVER_PUBLIC_IP`
6. You will be asked for grader's password. To unable it, open configuration file again: `$ sudo nano /etc/ssh/sshd_config`
7. Change `PasswordAuthentication yes` to **no**
8. Restart SSH: `$ sudo service ssh restart`

## Configure the local timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Choose **None of the above** to set timezone to UTC

## Install and configure Apache
1. Install **Apache**: `$ sudo apt-get install apache2`
2. Go to http://18.218.99.181/, if Apache is working correctly, a **Apache2 Ubuntu Default Page** will show up

## Install and configure Python mod_wsgi
1. Install the **mod_wsgi** package: `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable **mod_wsgi**: `$ sudo a2enmod wsgi`
3. Restart **Apache**: `$ sudo service apache2 restart`
4. Check if Python is installed: `$ python`

## Install PostgreSQL
1. Run `$ sudo apt-get install postgresql`
2. Make sure PostgreSQL does not allow remote connections
3. Open file: `$ sudo nano /etc/postgresql/10/main/pg_hba.conf`
4. Check to make sure it looks like this:
   ```
   # Database administrative login by Unix domain socket
   local   all             postgres                                peer

   # TYPE  DATABASE        USER            ADDRESS                 METHOD

   # "local" is for Unix domain socket connections only
   local   all             all                                     peer
   # IPv4 local connections:
   host    all             all             127.0.0.1/32            md5
   # IPv6 local connections:
   host    all             all             ::1/128                 md5
   ```
## Create new PostgreSQL user called **catalog**
1. Switch to PostgreSQL defualt user **postgres**: `$ sudo su - postgres`
2. Connect to PostgreSQL: `$ psql`
3. Create user **catalog** with LOGIN role: `# CREATE ROLE catalog WITH PASSWORD 'catalog';`
4. Allow user to create database tables: `# ALTER USER catalog CREATEDB;`
5. Create database: `# CREATE DATABASE catalog WITH OWNER catalog;`
6. Connect to database **catalog**: `# \c catalog`
7. Revoke all the rights: `# REVOKE ALL ON SCHEMA public FROM public;`
8. Grant access to **catalog**: `# GRANT ALL ON SCHEMA public TO catalog;`
9. Exit psql: `\q`
10.Exit user **postgres**: `exit`

## Create new Linux user called **catalog** and new database
1. Create a new Linux user: `$ sudo adduser catalog`
2. Give **catalog** user sudo access:
   * `$ sudo visudo`
   * Add `$ catalog ALL=(ALL:ALL) ALL` under line `$ root ALL=(ALL:ALL) ALL`
   * Save and exit the file
3. Log in as **catalog**: `$ sudo su - catalog`
4. Create database **catalog**: `createdb catalog`
5. Exit user **catalog**: `exit`

## Install git and clone catalog application from github
1. Run `$ sudo apt-get install git`
3. CD to www directory: `$ cd /var/www/`
4. Clone the catalog app: `$ sudo git clone GIT_PROJECT_UR `
5. Change the ownership: `$ sudo chown -R ubuntu:ubuntu yourproject/`
6. CD to `/var/www/yourproject`
7. Change file **application.py** to **__init__.py**: `$ mv application.py __init__.py`
8. Change line `app.run(host='0.0.0.0', port=8000)` to `app.run()` in **__init__.py** file

## Edit client_secrets.json file
1. Create a new project on Google API Console and download `client_scretes.json` file
2. Copy and paste contents of downloaded `client_scretes.json` to the file with same name under directory `/var/www/yourprojecr/client_secrets.json`
same for facebook client secret


## Setup for deploying a Flask App on Ubuntu VPS
1. Install pip: `$ sudo apt-get install python-pip`
2. Install packages:
```
   $ sudo pip install httplib2
   $ sudo pip install requests
   $ sudo pip install --upgrade oauth2client
   $ sudo pip install sqlalchemy
   $ sudo pip install flask
   $ sudo apt-get install libpq-dev
   $ sudo pip install psycopg2
   ```

## Setup and enble a virtual host
1. Create file: `$ sudo touch /etc/apache2/sites-available/yourproject.conf`
2. Add the following to the file:
```
   <VirtualHost *:80>
		ServerName XX.XX.XX.XX
		ServerAdmin admin@xx.xx.xx.xx
		WSGIScriptAlias / /var/www/yourproject/yourproject.wsgi
		<Directory /var/www/yourproject/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		Alias /static /var/www/yourproject/static
		<Directory /var/www/static/>
			Order allow,deny
			Allow from all
			Options -Indexes
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
   </VirtualHost>
   ```
3. Run `$ sudo a2ensite yourproject` to enable the virtual host
4. Restart **Apache**: `$ sudo service apache2 reload`

## Configure .wsgi file
1. Create file: `$ sudo touch /var/www/yourproject/yourproject.wsgi`
2. Add content below to this file and save:
```
   #!/usr/bin/python
   import sys
   import logging
   logging.basicConfig(stream=sys.stderr)
   sys.path.insert(0,"/var/www/yourproject/")

   from __init__ import app as application
   application.secret_key = 'super_secret_key'
```
3. Restart **Apache**: `$ sudo service apache2 reload`

## Edit the database path
1. Replace lines in `__init__.py`, `database_setup.py`, and `tvshows.py` with `engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost')`

## Disable defualt Apache page
1. `$ sudo a2dissite 000-defualt.conf`
2. Restart **Apache**: `$ sudo service apache2 reload`

## Set up database schema
1. Run `$ sudo python database_setup.py`
2. Run `$ sudo python tvshows.py` to populate database
3. Restart **Apache**: `$ sudo service apache2 reload`
4. Now follow the link to http://35.181.55.110/  the application should be runing online
5. If internal errors occur: check the [Apache error file](https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files)
note : if you get a server error when visiting run the command `sudo tail /var/log/apache2/error.log
` to see the errors and correct them
## Sources
1. [Amazon Lightsail Website](https://aws.amazon.com/lightsail/?p=tile)
2. [Google API Concole](https://console.cloud.google.com/)
3. [Udacity](https://www.udacity.com)
4. [Apache](https://httpd.apache.org/docs/2.2/configuring.html)
5. [Mr. google](https://google.com) ^_^
