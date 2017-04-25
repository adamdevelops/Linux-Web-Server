# Linux-Web-Server

## Objective:

Setup a Linux server to serve the Item-Catalog application we created earlier in the Full Stack Nanodegree course. Once the server is setup, we will secure it through addressing file permissions on the application files, configuring the ports on firewall (within Lightsail and inside the VM), and securing connection to the database.

## Step by Step Walkthrough:

### Get your server up and running
1. Create a Lightsail server instance of Ubuntu on Amazon Lightsail.
2. Within Lightsail in the account page, download the private key and place it directory named '.ssh' (If you do not have the folder located in your "C:\Users \ <username> \ .ssh", then create the folder with that name)

### Secure your server

### Changing timezone to UTC
1. Type in the following command:
`sudo dkpg-reconfigure tzdata`
2. A GUI should appear asking you to select a geographic area, select 'None of the above'.
3. You should another dialog asking to select a city/region corresponding to your time-zone. Scroll down the list until you see 'UTC' and press OK.
