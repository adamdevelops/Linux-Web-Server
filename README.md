# Linux-Web-Server

## Objective:

Setup a Linux server to serve the Item-Catalog application we created earlier in the Full Stack Nanodegree course. Once the server is setup, we will secure it through addressing file permissions on the application files, configuring the ports on firewall (within Lightsail and inside the VM), and securing connection to the database.

## Step by Step Walkthrough:

### Get your server up and running
1. Create a Lightsail server instance of Ubuntu on Amazon Lightsail.
2. Within Lightsail in the account page, download the private key and place it directory named '.ssh' (If you do not have the folder located in your "C:\Users \ username \ .ssh", then create the folder with that name)
<br>`cd  ~/.ssh
mv ~/Downloads/LightsailDefaultPrivateKey.pem ~/.ssh`
3. Then run the following command to SSH into your LightSail VM
`ssh ubuntu@52.71.200.120 -i LightsailDefaultPrivateKey.pem`


### Secure your server

### Changing timezone to UTC
<a href="http://stackoverflow.com/questions/22853026/ubuntu-change-timezone-to-utc-does-not-affect-the-time-of-syslog">Source</a>
1. Type in the following command:
`sudo dkpg-reconfigure tzdata`
2. A GUI should appear asking you to select a geographic area, select 'None of the above'.
3. You should another dialog asking to select a city/region corresponding to your time-zone. Scroll down the list until you see 'UTC' and press OK.
