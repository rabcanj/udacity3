# Linux Server Configuration

This document contains instructions that were applied to deploy Flask application from the previous project. In the bottom you can find rubics table summary where I summarized how I did specific tasks for this project.

Grader ssh keys are located in grader-ssh directory. File grader_key, which is a private key to login via ssh.

## Server

For this project Amazon Lightsail has been recommended. Unfortunately, Amazon Lightsail requires credit/debt card number even for free tier. Amazon does not accept Maestro cards at all, and since I do not have another card I decided to use VPS from DigitalOcean with Ubuntu 18.04.

### Server details:
- IP Adress: 165.22.94.80
- URL: [https://165.22.94.80.xip.io/](#)

### Summary of software installed

- Git
- Nginx
- Virtualenv
    - Gunicorn
    - Flask
    - sqlalchemy
    - Jinja2
    - flask_oauthlib
    - psycopg2-binary
- Postgres

### Summary of Configurations Made

#### Initial Configuration

1. Create an account on DigitalOcean and create new VPS server - Droplet (Droplet costs 5 USD per month).
1. The VPS was creatd on IP: 165.22.94.80. Now we can login to VPS server from our local machine, do so with the following command: `shh root@165.22.94.80`
1.  The SSH default port is 22. We need to change it to 2200. Edit file */etc/ssh/sshd_config*, change line 13 from `#Port 22` to `Port 2200`. After that restart SSH: `sudo service ssh restart`
1. The login command from step 2 does not work anymore, now you must log in with the following comand: `shh root@165.22.94.80 -p 2200`


#### Upgrade packages

One of the requirements to finish this project is that the installed packages need to be up to date. We do so with the following commands: `sudo apt-get update && sudo apt-get upgrade`. Also check if timezone is UTC, if not change it to UTC.


#### Create Grader with sudo permissions

1.  Run `sudo add user grader`
1.  Enter password for grader, I used: ovvidowa
1.  Run `sudo visudo`
1.  Add there the following line: `grader  ALL=(ALL:ALL) ALL`
1.  Now you can switch from root to grader with the following command: `su - grader`


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

#### Setup SSH for grader and disable SSH for root

1.  Create new pair of keys that will be used for grader, you can use ssh-keygen on your local machine
1.  Log in into VPS server and switch account to grader
1.  In /home/grader create .ssh folder
1.  In /home/grader/authorized_keys paste generated public key from step 1
1.  In your local machine copy private key <keyname> to your .ssh directory
1.  On VPS server open /etc/ssh/sshd_config and check if `PasswordAuthentification`is equal to `no` and `PermitRootLogin` is also equal to no. This ensure that you cannot log as a root remotely and you must use ssh
1.  sudo service ssh restart
1.  Now you can login as a grader via ssh:

        ssh -i ~/.ssh/<keyname> -p 2200 grader@165.22.94.80
Note that `ssh -p 2200 root@165.22.94.80` does not work anymore, says root@165.22.94.80: Permission denied (publickey).


#### Git

1. Install Git: `sudo apt-get install git`
1. Setup git emial, username
1. Clone catalog app 'git clone https://github.com/rabcanj/udacity2.git'

#### Postgres

The previous project was using sqllite, we need to change to postgresql. Change database engine to (in file controlers.py and database.py): `db_engine='postgresql://catalog:ovviodwa@localhost/catalog`

  ##### Install Postgres

  1.  Install Postgres: `sudo apt-get install postgresql`
  1.  run: `psql`
  1.  run: `CREATE ROLE catalog WITH LOGIN;`
  1.  run: `ALTER ROLE catalog CREATEDB;`
  1.  run: `\password catalog` -> ovviodwa
  1.  run: `\q`
  1.  run: `exit`


  ##### Create new database and user called Catalog
  1.  run: `sudo add user catalog`
  1.  run: `sudo visudo`
  1.  Add line `catalog ALL=(ALL:ALL) ALL`
  1.  Run: `sudo su - catalog`
  1.  run: `createdb catalog`
  1.  Run: `exit`


#### Gunicorn

As a WSGI server I decided to use Gunicorn. To be able to use gunicorn follow this steps:
1.  Open project Item Catalog and add to the requirements.txt Gunicorn.
1.  Create new file `wsgi.py` and paste there following content

        from app import app as application
        from project import controlers, models, fboauth, database

        if __name__ == "__main__":
          application.run()
1. Create new file `runserver_wsgi.sh` and paste there the following content:

        gunicorn --bind 'unix:/home/grader/udacity2/catalog.sock' wsgi:application --deamon

#### Nginx

As a web server I decided to use Nginx. To configure nginx follow the next steps:

1.  Install Nginx: `sudo apt-get install nginx`
1.  `cd /etc/nginx/sites-available`
1.  Create new file `catalog`
1.  Paste the following content to the catalog:

        server {
          listen 443 ssl;
          ssl_certificate     /home/grader/udacity2/dev.crt;
          ssl_certificate_key /home/grader/udacity2/dev.key;

          location / {
              include proxy_params;
              proxy_pass http://unix:/home/grader/udacity2/somename.sock;
          }
        }
1. Create new link to the sites-enabled: `sudo ln -s /etc/nginx/sites-available/catalog /etc/nginx/sites-enabled`
1. We do not need to serve default nginx app, so remove it: `cd /etc/nginx/sites-enabled && rm default`
1. Start the application: `source /home/grade/udacity2/runserver_wsgi.sh `

### Conclusion

In this project I configured web server where a simple web app is running. Insetad of port 80 the port 443 was used, because facebook login does not work with 80.

| Criteria   |            |
|----------|:-------------:|
|Can you log into the server as the user grader using the submitted key? | ssh -i ~/.ssh/grader_key -p 2200 grader@165.22.94.80 |
| Is remote login of the root user disabled? |   root login disabled in /etc/ssh/sshd_config   |  
| Is the grader user given sudo access? | sudo visudo, added line grader  ALL=(ALL:ALL) ALL |  
| Is the firewall configured to only allow for SSH, HTTP, and NTP? | firewall allwos SSH on 2200, HTTPS(443), NTP(123), server runs on https, so http was not allowed|  
| Are users required to authenticate using RSA keys?| password login disabled in /etc/ssh/sshd_config |  
| Are the applications up-to-date? | sudo apt-get update && sudo apt-get upgrade |
| Is SSH hosted on non-default port? | tcp/2200 |
| Is there a web server running on port 80? | webserver is running on 443 because instead of http I used https |
| Has the database server been configured to properly serve data? | As a database server postgresql is used (see section about postresql) |
| Has the web server been configured to serve the Item Catalog application? | Item Catalog application is served by Gunicorn (wsgi) and Nginx |
| Is a README file included in the GitHub repo containing all specified information? | I believe yes |


### List of third party resources

1.  https://www.digitalocean.com/community/tutorials/how-to-serve-flask-applications-with-uswgi-and-nginx-on-ubuntu-18-04

1.  https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04

1.  http://docs.gunicorn.org/en/stable/deploy.html

1.  https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-18-04

1.  https://www.ssh.com/ssh/keygen/

1.  https://vitux.com/add-and-manage-user-accounts-in-ubuntu-18-04-lts/

1.  https://developers.facebook.com/blog/post/v2/2017/12/18/strict-uri-matching/

1.  https://stackoverflow.com/questions/37001004/facebook-login-message-url-blocked-this-redirect-failed-because-the-redirect

1.  https://pythonhosted.org/Flask-Social/
