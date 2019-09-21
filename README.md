# Linux Server Configuration

This document contains instructions that were applied to deploy Flask application from the previous project.

## Server

For this project Amazon Lightsail has been recommended. Unfortunately, Amazon Lightsail requires credit/debt card number even for free tier. Amazon does not accept Maestro cards at all, and since I do not have another card I decided to use VPS from DigitalOcean with Ubuntu 18.04.

### Summary of Server Configurations made

#### Initial Configuration

1. Create an account on DigitalOcean and create new VPS server - Droplet (Droplet costs 5 USD per month).
1. The VPS was creatd on IP: 165.22.94.80. Now we can login to VPS server from our local machine, do so with the following command: `shh root@165.22.94.80`
1.  The SSH default port is 22. We need to change it to 2200. Edit file */etc/ssh/sshd_config*, change line 13 from `#Port 22` to `Port 2200`. After that restart SSH: `sudo service ssh restart`
1. The login command from step 2 does not work anymore, now you must log in with the following comand: `shh root@165.22.94.80 -p 2200`


 #### Upgrade packages

One of the requirements to finish this project is that the installed packages need to be up to date. We do so with the following commands:

      sudo apt-get update
      sudo apt-get upgrade


#### Create Grader with sudo permissions

1.  Run `sudo add user grader`
1.  Enter password for grader, I used: ovvidowa
1.  Run `sudo visudo`
1.  Add there the following line: `grader  ALL=(ALL:ALL) ALL`
1.  Now you can log as grader with the command: `su - grader`


#### Firewall Configuration

Ubuntu provides preinstalled firewall `ufw`. To configure it follow this steps:
1.  By default the firewall is dissabled. Enable it with the command: `sudo ufw enable`
1.  Deny all incoming communication: `sudo ufw default deny incoming`
1.  Allow all outgoing communication: `sudo ufw default allow outgoing`
1.  Allow ssh communication: `sudo ufw allow ssh`
1.  Allow communication on the port 2200, so that ssh on 2200 will work: `sudo ufw allow 2200/tcp`
1.  Allow http server: `sudo ufw allow www`
1.  Allow https server: `sudo ufw allow 443/tcp`
1.  Allow NTP: `sudo ufw allow 123/udp`
1.  Denny port 22: `sudo ufw deny 22`
1.  Chcek status: `sudo ufw status`
