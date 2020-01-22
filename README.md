# Linux-Server-Configuration
Final Project: Linux Server Configuration

## Amazon Lightsail Server Set Up

1. [Visit Amazon Lightsail](https://signin.aws.amazon.com/signin?redirect_uri=https%3A%2F%2Flightsail.aws.amazon.com%2Fls%2Fwebapp%3Fstate%3DhashArgs%2523%26isauthcode%3Dtrue&client_id=arn%3Aaws%3Aiam%3A%3A015428540659%3Auser%2Fparksidewebapp&forceMobileApp=0) and create an AWS account:

2. After you log in, click 'Create Instance';

3. Select Platform (linux):
![create instance](https://github.com/mike3983/Linux-Server-Configuration/blob/master/pics/35C4D8EE-512D-4B9B-9474-1C81AC87441F.jpeg)

4. Select Blueprint (Ubuntu). Scroll down to name your instance. Click 'Create':
![select blueprint](https://github.com/mike3983/Linux-Server-Configuration/blob/master/pics/5ED3AAE2-FD3A-4CCE-BA1E-77F00D4267D6.jpeg)

5. The instance needs about 2 mins to set up. After it is set up, you will see 'running' in the left corner of the status card. Take note of the public IP.

6. Click the status card and navigate to account

7. Download your private key which is a .pem file.
![download pem key](https://github.com/mike3983/Linux-Server-Configuration/blob/master/pics/FB9884C5-F333-4FB1-B23A-C92AF85C0890.jpeg)

8. Click the 'Networking' tab and find the 'Add another' at the bottom. Add port 123 (udp) and 2200(tcp).
![modify ports](https://github.com/mike3983/Linux-Server-Configuration/blob/master/pics/046BB102-3535-44CE-80A0-944312457032.jpeg)

## Server Configuration

1. Save the downloaded `.pem` public key file into .ssh folder which is based in the home directory of your personal computer

2. Secure public key while also making it accessible `$ chmod 600 ~/.ssh/MyAWSKey.pem`  ** From here, take note of the use of first and second instances of terminals. The first terminal will connect to the AWS server; the second terminal will be on the personal computer and used to create the grader's account

3. Open the first terminal and use this key to log into our Amazon Lightsail Server: `$ ssh -i ~/.ssh/MyAWSKey.pem ubuntu@18.191.85.141`

4. Log in as root user`$ sudo su -`

5. Then type  `$ sudo adduser grader` to create another user 'grader'

6. Create a new file in the sudoers directory: `$ sudo nano /etc/sudoers.d/grader`. And give grader the super permisssion `grader ALL=(ALL:ALL) ALL`. Save and exit

7. Run the following commands to update all packages and install finger package:
- `$ sudo apt-get update`
- `$ sudo apt-get upgrade`
- `$ sudo apt-get install finger`

8. Open a new(now the second) Terminal window (Command+N) and input `$ ssh-keygen -f ~/.ssh/udacity_key.rsa`

9. Stay on the same(i.e the second) Terminal window, input `$ cat ~/.ssh/udacity_key.rsa.pub` to read the public key. Copy the public key.

10. Return to the first terminal window where you are logged into Amazon Lightsail as the root user, move to grader's folder by `$ cd /home/grader`

11. Create a .ssh directory (in same first terminal): `$ mkdir .ssh`

12. Create a file to store the public key(still in first terminal...use of first terminal will end in #18): `$ touch .ssh/authorized_keys`

13. Edit the authorized_keys file `$ nano .ssh/authorized_keys`

14. Change the permission: `$ sudo chmod 700 /home/grader/.ssh` and `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`

15. Change the owner from root to grader: `$ sudo chown -R grader:grader /home/grader/.ssh`

16. Restart the ssh service: `$ sudo service ssh restart`

17. Type `exit` to disconnect from Amazon Lightsail server

18. Log into the server as grader(that is log in via the second terminal window): `$ ssh -i ~/.ssh/udacity_key.rsa grader@18.191.85.141`

19. Enforce the key-based authentication: `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and change text to `no`. After this, restart ssh again: `$ sudo service ssh restart`

20. Change the ssh port from 22 to 2200: `$ sudo nano /etc/ssh/ssdh_config` Find the *Port* line and change `22` to `2200`. Restart ssh: `$ sudo service ssh restart`

21. Disconnect the server by `^c` and then log back through port 2200: `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.58.109.116` now loggin in via port 2200

22. Disable ssh login for *root* user to prevent attackers from making a fondering attept with root: `$ sudo nano /etc/ssh/sshd_config`. Find the *PermitRootLogin* line and edit to `no`. Restart ssh `$ sudo service ssh restart`

23. Configure the Firewall:
- `$ sudo ufw status`
- `$ sudo ufw allow 2200/tcp`
- `$ sudo ufw allow 80/tcp`
- `$ sudo ufw allow 123/udp`
- `$ sudo ufw enable`

## Deploy Catalog Application

Ensure you are logged in as grader. Should at anypoint a ubuntu password is requested simply ^d and use `sudo` to re-execute that command.

1. Install required packages
- `$ sudo apt-get install apache2`
- `$ sudo apt-get install libapache2-mod-wsgi-py3 libapache2-mod-wsgi python-dev`
- `$ sudo apt-get install git`
- `$ sudo apt-get install python-pip`

2. Enable mod_wsgi `$ sudo a2enmod wsgi` and start the web server by `$ sudo service apache2 start` or `$ sudo service apache2 restart`

3. Enter your public IP address in your browser now and if apache2 installed correctly you should this page in your browser:
![apache page](https://github.com/mike3983/Linux-Server-Configuration/blob/master/pics/A51C4146-56E4-4003-9B7D-420E3F75CD9D.jpeg)

4. Create catalog folder to keep app and make grader owner and group of the folder
- `$ cd /var/www`
- `$ sudo mkdir WebApp`
- `$ sudo chown -R grader:grader WebApp`
- `$ cd WebApp`

5. Clone the project from Github: `$ git clone [your link] WebApp` (so folder path to app will become `var/www/WebApp/WebApp`). All required files must be in the WebApp folder.
![clone github](https://github.com/mike3983/Linux-Server-Configuration/blob/master/pics/86EFC227-2B80-43BE-A8DC-F567A658645B_4_5005_c.jpeg)

6. Create a .wsgi file in `/var/www/WebApp/`: `$sudo nano webapp.wsgi` and add the following code into this file:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/WebApp/")

from WebApp import app as application
application.secret_key = 'super_secret_key'
```

7. In /var/www/WebApp/WebApp Rename the `application.py` to `__init__.py` as follows `mv application.py __init__.py`

8. Install the Flask and other packages needed for this application
- `$ sudo pip install Flask`
- `$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests`

9. Configure and enable the virtual host: `$ sudo nano /etc/apache2/sites-available/catalog.conf`
Paste the following code and save:
```
<VirtualHost *:80>
                ServerName 18.191.85.141
                ServerAdmin email@mywebsite.com
                ServerAlias ec2-18-191-85-141.us-east-2.compute.amazonaws.com
                WSGIScriptAlias / /var/www/WebApp/webapp.wsgi
                <Directory /var/www/WebApp/WebApp/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/WebApp/WebApp/static
                <Directory /var/www/WebApp/WebApp/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
You can find your host name in this link: http://www.hcidata.info/host2ip.cgi

## Setup Database

1. Now we need to set up the database
- `$ sudo apt-get install libpq-dev python-dev`
- `$ sudo apt-get install postgresql postgresql-contrib`
- `$ sudo -u postgres -i`

2. Login as user "postgres" `$ sudo su - postgres`
You should see the username changed again in command line, and type `$ psql` to get into postgres command line

3. Create a new database named 'soc' and create a new user named 'mikeg' in postgreSQL shell

- `postgres=# CREATE DATABASE soc;`
- `postgres=# CREATE USER mikeg;`

4. Set a password for user catalog

`postgres=# ALTER ROLE mikeg WITH PASSWORD '';`

5. Give user "mikeg" permission to "soc" application database

`postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`

6. Quit postgreSQL -`postgres=# \q`

Exit from user "postgres"

`exit`

## Adjust Python Files and Launch App

1. Use `sudo nano` command to change all engine to `engine = create_engine('postgresql://mikeg:[your password]@localhost/soc)` e.g engine = create_engine('engine = create_engine('postgresql://mikeg:[your password]@localhost/soc')
Base.metadata.bind = engine. 

Perform this same step in the database_setup.py and the dbdata.py files.

2. Initiate the database: `$ python database_setup.py` Populate the database by running: `$ python dbdata.py`

3. Restart Apache server `$ sudo service apache2 restart` and enter your public IP address or host name into the browser. The web application should be online now!


## Appendix:

1. [Public IP address:](18.191.85.141)

2. GitHub Item Catalog Repository: https://github.com/mike3983/Item-Catalog

3. NOTE: During the grading process, requested to perform `$ sudo ufw deny 22` to disable port 22


## Reference

1. How to Deploy a Flask App to a Linux Server - https://www.youtube.com/watch?v=YFBRVJPhDGY
2. https://github.com/anumsh/Linux-Server-Configuration
3. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
4. https://github.com/juvers/Linux-Configuration/blob/master/README.md
