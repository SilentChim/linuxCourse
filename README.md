# linuxCourse

Udacity Full Stack Nanodegree - Linux Server Configuration Project

Host Name: ec2-18-217-189-214.us-east-2.compute.amazonaws.com

IP Address: 18.217.189.214

### Server Configuration

Click the status card and you will get into this page. Click the Account page at the bottom account page

Click the 'Download' to download your private key, it should go to your Download folder by default. It is a .pem file, not the rsa file in other step-by-step guides, but we can still use it to log into our server private key

Click the 'Networking' tab and find the 'Add another' at the bottom. Add port 123 and 2200. Amazon Lightsail allows only port 22 and 80 by default, no matter how you set it up in ubuntu's ufw AWS Firewall

Server Configuration
Move your downloaded .pem public key file into .ssh folder that is at the root of Finder move private key

We need to make our public key usable and secure.

`$ chmod 600 ~/.ssh/YourAWSKey.pem`

We now use this key to log into our Amazon Lightsail Server: 

`$ ssh -i ~/.ssh/YourAWSKey.pem ubuntu@[YOUR PUBLIC IP ADDRESS]`

Become root user:

`$ sudo su -`

Create another user 'grader' root user

`$ sudo adduser grader` to create another user 'grader' root user

Now create a new file under the sudoers directory: 

`$ sudo nano /etc/sudoers.d/grader`

Fill that file with `grader ALL=(ALL) NOPASSWD:ALL`, then save it.

Update all packages and install finger package:

`$ sudo apt-get update`

`$ sudo apt-get upgrade`

`$ sudo apt-get install finger`

Open new Terminal window and create public key:

`$ ssh-keygen -f ~/.ssh/udacity_key.rsa`

Read and copy the public key in same window: 

`$ cat ~/.ssh/udacity_key.rsa.pub`

In root user terminal, change directory to grader's folder:

`$ cd /home/grader`

Create a .ssh directory: 

`$ mkdir .ssh`

Create a file to store the public key: 

`$ touch .ssh/authorized_keys`

`$ nano .ssh/authorized_keys`

Change the permissions:

`$ sudo chmod 700 /home/grader/.ssh`

`$ sudo chmod 644 /home/grader/.ssh/authorized_keys`

Transfer ownership from root to grader:

`$ sudo chown -R grader:grader /home/grader/.ssh`

Restart the ssh service:

`$ sudo service ssh restart`

Type `$ ~.` to disconnect from AWS server

Log into the server as grader:

`$ ssh -i ~/.ssh/udacity_key.rsa grader@[YOUR PUBLIC IP ADDRESS]`

Enforce the key-based authentication:
`$ sudo nano /etc/ssh/sshd_config`

Find PasswordAuthentication and change to no.

Restart ssh again:

`$ sudo service ssh restart`

Change ssh port from 22 to 2200:

`$ sudo nano /etc/ssh/ssdh_config`

Find the Port line and change 22 to 2200. 

Restart ssh:

`$ sudo service ssh restart`

Disconnect the server: 

`$ ~.` 

Log back in:

`$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@[YOUR PUBLIC IP ADDRESS]`

Disable ssh login for root user:

`$ sudo nano /etc/ssh/sshd_config`

Change PermitRootLogin to no.

Restart:

`ssh $ sudo service ssh restart`

Configure UFW:

`$ sudo ufw allow 2200/tcp`
`$ sudo ufw allow 80/tcp`
`$ sudo ufw allow 123/udp`
`$ sudo ufw enable`

### Deploy Catalog Application
Install required packages

`$ sudo apt-get install apache2`

`$ sudo apt-get install libapache2-mod-wsgi python-dev`

`$ sudo apt-get install git`

Enable mod_wsgi: 

`$ sudo a2enmod wsgi`

`$ sudo service apache2`

`$ sudo service apache2 restart`

Create directory structures:

`$ cd /var/www`
`$ sudo mkdir catalog`
`$ sudo chown -R grader:grader catalog`
`$ cd catalog`

Clone GitHub Repo:

`$ git clone [your link] catalog`

Create .wsgi file: 

`$sudo nano catalog.wsgi` 

Add following to this file:

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
Rename the application.py to __init__.py
```

Install and start the virtual environment:

`$ sudo pip install virtualenv`

`$ sudo virtualenv venv`

`$ source venv/bin/activate`

`$ sudo chmod -R 777 venv`

Install dependencies:

`$ sudo apt-get install python-pip`

`$ sudo pip install Flask`

`$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 requests redirect`

Change client_secrets.json:

`nano __init__.py` 

Change `client_secrets.json` to `/var/www/catalog/catalog/client_secrets.json` and change the host to your Amazon Lightsail public IP address and port to 80

Configure and enable the virtual host

`$ sudo nano /etc/apache2/sites-available/catalog.conf`

Paste the following code and save:

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
Create database:

`$ sudo apt-get install libpq-dev python-dev`

`$ sudo apt-get install postgresql postgresql-contrib`

`$ sudo -u postgres -i`

Type `$ psql` to get into psql command line

Create a user to set up the database:

`$ CREATE USER catalog WITH PASSWORD [your password];`

`$ ALTER USER catalog CREATEDB;`

`$ CREATE DATABASE catalog WITH OWNER catalog;`

Connect to database:

`$ \c catalog`

`$ REVOKE ALL ON SCHEMA public FROM public;`

`$ GRANT ALL ON SCHEMA public TO catalog;`

Exit the postgres command line

Use `sudo nano` command to change all engine to `engine = create_engine('postgresql://catalog:[your password]@localhost/catalog`

Initiate database script:

`db_setup.py`

Restart Apache server:

`$ sudo service apache2 restart` and enter your public IP address or host name into the browser

References:

http://www.hcidata.info/host2ip.cgi
https://www.youtube.com/watch?v=HcwK8IWc-a8
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
