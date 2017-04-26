# Linux-Web-Server

## Objective:

Setup a Linux server to serve the Item-Catalog application we created earlier in the Full Stack Nanodegree course. Once the server is setup, we will secure it through addressing file permissions on the application files, configuring the ports on firewall (within Lightsail and inside the VM), and securing connection to the database.

## Step by Step Walkthrough:

### <u>Get your server up and running:</u>
1. Create a Lightsail server instance of Ubuntu on Amazon Lightsail.
2. Within Lightsail in the account page, download the private key and place it directory named '.ssh' (If you do not have the folder located in your "C:\Users \ username \ .ssh", then create the folder with that name)
```
$ cd  ~/.ssh
$ mv ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh
```
3. Then run the following command to SSH into your LightSail VM
<br>`
ssh ubuntu@52.71.200.120 -i LightsailDefaultPrivateKey.pem
`

Note: This command won’t work later when changing ssh access during the firewall configuration.

### <u>Secure your server:</u>
Now that we are SSH'd into our web server. We will need to secure it as well as install various packages to deploy our project on the server.

#### Update and upgrade system packages
1. Update and upgrade the current system packages in your linux server instance using the following commands.
```
sudo apt-get update
sudo apt-get upgrade
```

#### Configuring the Firewall
Configuring the Lightsail firewall
1. On the webpage for the Lightsail console for your server, go to the Networking tab and edit the rules for the firewall to allow SSH connections via port 2200 by adding another rule with the definition of a 'Custom' application using application 'TCP' on a port range of '2200'.

Configuring the Uncomplicated Firewall (UFW)
These are the folowing commands that you will enter to configure the UFW within your linux web server instance.
1. Deny by default all the incoming connections
<br>`
sudo ufw default deny incoming
`
2. Allow by default any outgoing connections
<br>`
sudo ufw default allow outgoing
`
3. Allow SSH connections through the UFW
<br>`
sudo ufw allow ssh
`
4. Allow TCP connections via port 2200 through the UFW
<br>`
sudo ufw allow 2200/tcp
`
5. Allow HTTP connections through the UFW
<br>`
sudo ufw allow www
`
6. Allow SSH connections through the UFW
<br>`
sudo ufw enable
`
7. Now the firewall with the added rules should be enabled, we can view our changes with the following command:
<br>`
sudo ufw status
`

#### Giving grader account access to webserver
1. Create a new uer account called 'grader'. Provide the account an password when prompted and fill out the user information such as Full name (not necessary to fill out any of the other fields).
<br>`
sudo adduser grader
`
2. Give the grader account sudo access. This is accomplished by copying the sudoers rules for the root account for the grader account and editting the rules to apply for sudo access for the grader account
```
$ sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
Note: This is the account with sudo access may vary for you.

$ sudo nano /etc/sudoers.d/grader

Within nano, replace the user name that is there with grader
```

3. On your local machine, run this command through Git Bash to generate public/private ssh key pair. You will be prompted to a file to save the key (I went with the default route as displayed in the prompt's parentheses).
<br>`
$ ssh-keygen
`
4. After the ssh key has been made, we place the contents in the public SSH key into a directory on our server in a directory called authorized_keys.
```
On your server, create a .ssh/ directory in grader and another directory within .ssh/ named authorized_keys.

$ mkdir .ssh
$ touch .ssh/authorized_keys


Read the contents of the public key on your local machine in the .ssh directory (note that my key is named id_rsa, replace it with name you saved it as during keygen setup)
$ cat .ssh/id_rsa.pub

Copy the text displayed and paste it into the authorized_keys file on the server with the .ssh/ directory.

$ nano .ssh/authorized_keys

Set the file permissions on the .ssh directory as well as the authorized_keys file. 
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys

Change the owner of the .ssh directory to grader
$ sudo chown -R grader:grader /home/grader/.ssh

Finally, you should be able to SSH into your web server using the below command (replace the IP Address as the public IP Address of your own server listed on your Lightsail console webpage):
ssh grader@54.236.202.30 -i ~/.ssh/id_rsa -p 2200

    user  @  public IP       location of      port # 
                             private key
```
5. We will now configure our server to force key based authentication to login into the server and to only listen on port 2200 for login.
<br>`
$ sudo nano /etc/ssh/sshd_config
`<br>
Search for the line 'Password Authentication' and change 'yes' to 'no' to force key based authentication. At the beginning of this file under the text for ports we listen for, comment out 'Port 22' and type in 'Port 2200' under it to allow only port listening for port 2200.

#### Prepping project for deployment
Changing timezone to UTC
<br>
<a href="http://stackoverflow.com/questions/22853026/ubuntu-change-timezone-to-utc-does-not-affect-the-time-of-syslog">Source</a>
1. Type in the following command:
`sudo dkpg-reconfigure tzdata`
2. A GUI should appear asking you to select a geographic area, select 'None of the above'.
3. You should another dialog asking to select a city/region corresponding to your time-zone. Scroll down the list until you see 'UTC' and press OK.

#### Installing and configuring Apache to serve a Python mod_wsgi application
1. Install Apache.
<br>`
sudo apt-get install apache2
`
2. Install the Apache mod_wsgi package
<br>`
sudo apt-get install libapache2-mod-wsgi
`

#### Installing and configuring PostgreSQL
1.Install PostgreSQL:
<br>`
sudo apt-get install postgresql postgresql-contrib
`<br>
2. Verify if remote connections are disabled by default in the pg_hba.conf within the lines pertaining to the only allowed connections from the local host addresses 127.0.0.1 for IPv4 and ::1 for IPv6.
<br>`
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
`<br>
3. Create a PostgreSQL user called 'catalog'
<br>`
sudo -u postgres createuser -P catalog
`<br>
4. Create an empty database called 'catalog' with the owner of user 'catalog' 
<br>`
sudo -u postgres createdb -O catalog catalog
`<br>

#### Installing Flask, SQLAlchemy, etc
1. Install the Flask, SQLAlchemy, and any other packages used in your Catalog project
```
sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
```

#### Installing Git
1. Install Git
`
sudo apt-get install git
`

#### Clone the repository for the Item Catalog Project
1. Create a directory in which to clone your Item Catalog project to from your GitHub.
```
cd /var/www/
sudo mkdir /catalog
git clone https://github.com/adamdevelops/Item-Catalog---DVD-Catalog.git
```

#### Create the catalog.wsgi file for the project
1. Create a 'catalog.wsgi' file with in your project folder with the following (the system path will be where you stored your project, and the application secret key will be what you set it to):
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog")

from catalog import app as application
application.secret_key = 'my secret key'  # This needs to be changed, used as an example
```

#### Configure Apache2 to serve the app
1. Create a virtual host configuration file in the '\etc\apache2\sites-available' directory in order to serve the catalog app using Apache web server. Here is the content of the file:
```
<VirtualHost *:80>
    ServerName 54.236.202.30
    ServerAlias ec2-54-236-202-30.us-east-1a.compute.amazonaws.com
    ServerAdmin admin@54.236.202.30
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog/catalog.wsgi
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
2. Disable the default virtual host with:
<br>`
sudo a2dissite 000-default.conf
`
3. Then enable the catalog app virtual host:
<br>`
sudo a2ensite catalog.conf
`
4. Restart Apache to make the configuration changes active:
<br>`
sudo service apache reload
`

Now the catalog app is active and can be access at:
` http://54.236.202.30 `

## Sources:
<br>
<u> Deploying a Flask app: </u>

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

<u>Securing remote access to database:<br>
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

https://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html

<u>Changing timezone: </u> <br>
http://stackoverflow.com/questions/22853026/ubuntu-change-timezone-to-utc-does-not-affect-the-time-of-syslog

<u>Installing Postgres: </u><br>
https://www.howtoforge.com/tutorial/ubuntu-postgresql-installation/#step-configure-postgresql-user

https://tecadmin.net/install-postgresql-server-on-ubuntu/#

http://hocuspokus.net/2008/05/install-postgresql-on-ubuntu-804/

https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04

<u>Creating user in database: </u><br>
https://www.postgresql.org/docs/8.0/static/sql-createuser.html

http://stackoverflow.com/questions/10861260/how-to-create-user-for-a-db-in-postgresql

https://www.postgresql.org/docs/8.0/static/sql-createuser.html

<u>Connecting as user in database: </u><br>
http://stackoverflow.com/questions/17443379/psql-fatal-peer-authentication-failed-for-user-dev

http://alvinalexander.com/blog/post/postgresql/log-in-postgresql-database
