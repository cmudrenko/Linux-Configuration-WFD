# Linux Server Configuration

* Author: Chris Mudrenko
* Email: cmudrenk@asu.edu

## Project Description:

This a Linux Server Configuration project for Udacity Full Stack Web Development program.
This project takes a baseline installation of a Linux distribution on a virtual machine 
and have it host my WhatsForDinner - Item Catalog project.

## Server Info:

  * Server IP Address: 34.218.78.235
  * SSH port: 2200
  * Application URL: http://34.218.78.235

## Connect to AWS Lightsail Instance via SSH:

  * Go to the Account page on the website, and select the "SSH keys" tab.
  * Download the keys (default is pre-selected).
  * Rename the downloaded file lightsail_key.rsa, and move in the local folder 
  ~/.ssh (if there isn't, cd in the home directory and create the directory by inputing mkdir .ssh)
  * Type in the terminal chmod 600 ~/.ssh/lightsail_key.rsa to fix the file permissions.
  * Now the instance can be accessed by inputing in the terminal the command 
  ```
  ssh -i ~/.ssh/lightsail_key.rsa ubuntu@34.218.78.235 (for now the default port used is 22).
  ```  

## Update all currently installed packages as "ubuntu" user:
  ```
    sudo apt-get update
    sudo apt-get upgrade
   ```  

## Add a new user "grader":
  ```
    sudo adduser grader
  ```
  * Give user name as "Udacity Grader" and password as "password"
  
## Grant sudo permission to user "grader":

  * Add a new file named "grader" under "/etc/sudoers.d/" directory
  ```
    sudo nano /etc/sudoers.d/grader
  ```
  * Type "grader ALL=(ALL:ALL) ALL" in the grader file, save and quit.

## Check that user "grader" has sudo permission:

    ```
      sudo cat /etc/passwd
    ```
    If the above command will show the contents of the file, grader user has sudo permission.

## Generate key for ssh login for user "grader":

  * On your local machine enter, ```ssh-keygen``` and save the private key in ~/.ssh/graderKey

  * On your virtual machine, switch the user to "grader" account and create .ssh directory and
  authorized_keys file in /home/grader/.ssh/authorized_keys:
  ```
    su - grader
    mkdir .ssh
    nano .ssh/authorized_keys
  ```
  * Copy the public key from local machine file named ".ssh/graderKey.pub" to 
  virtual machine file named ".ssh/authorized_keys" and save it.

  * Change the permission for directory and file .ssh/authorized_keys
  ```
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
  ```
  
  * Reload ssh and enter password:
  ```
    service ssh restart
  ```

## Login with ssh as user "grader" using key (default port is 22):

   ```
    ssh -i .ssh/graderKey grader@34.218.78.235
   ```

## Update all currently installed packages as "grader" user:
   ```
    sudo apt-get update
    sudo apt-get upgrade
   ```  

## Change SSH port from 22 to 2200 in the following file:
    ```
      sudo nano /etc/ssh/sshd_config
    ```
    - In the above file, change Port to 2200
    - Change PasswordAuthentication to no
    - Change PermitRootLogin to no
    - Save and quit the file.
    - Restart ssh service
    ```
      sudo service ssh restart
    ```
    - In AWS Lightsail Firewall Security, add 2200 as the inbound custom TCP Rule port.

## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

    ```
      sudo ufw default deny incoming
      sudo ufw default allow outgoing
      sudo ufw allow 2200/tcp
      sudo ufw allow 80/tcp
      sudo ufw allow 123/udp
      sudo ufw enable
    ```

## Check the status of Firewall configuration:
    ```
      sudo ufw status
    ```
    Status: active

    To                         Action      From
    --                         ------      ----              
    2200/tcp                   ALLOW       Anywhere                  
    80/tcp                     ALLOW       Anywhere                  
    123/udp                    ALLOW       Anywhere                             
    2200/tcp (v6)              ALLOW       Anywhere (v6)             
    80/tcp (v6)                ALLOW       Anywhere (v6)             
    123/udp (v6)               ALLOW       Anywhere (v6)    


## Confirm login with key as user grader to port 2200:
  ```
    ssh grader@34.218.78.235 -p 2200 -i ~/.ssh/graderKey
  ```
  
## Configure the local timezone to UTC

  ```
    sudo dpkg-reconfigure tzdata
  ```
  Timezone is already set to UTC

## Install Apache:
  ```
    sudo apt-get install apache2
  ```

## Install mod_wsgi:
  ```
    sudo apt-get install python-setuptools libapache2-mod-wsgi
  ```

## Restart Apache:
  ```
    sudo service apache2 restart
  ```

## Install and configure PostgreSQL:

  - Install PostgreSQL ```sudo apt-get install postgresql```
  - Login as user "postgres" ```sudo su - postgres```
  - Get to PostgreSQL shell: ```psql```
  - Create a new database and user name both named "menu":
    ```
      CREATE DATABASE menu;
      CREATE USER menu;
    ```
  - Set "menu" user's password: ```ALTER ROLE menu WITH PASSWORD 'password';```
  - Give "menu" user permission to "menu" application database
    ```
      GRANT ALL PRIVILEGES ON DATABASE menu TO menu;
    ```
  - Quit and exit postgreSQL:
    ```
      \q
      exit
    ```

## Install git, clone the WhatForDinner project and configure:
    ```
      sudo apt-get install git
      cd /var/www/
      sudo mkdir WFD
      sudo git clone https://github.com/cmudrenko/WFD.git WFD
      cd WFD
    ```
  - Make sure .git directory is not publicly accessible: ```sudo chmod 700 /var/www/WFD/WFD/.git```
  - In /var/www/WFD/WFD directory, change file name application.py to
  __init__.py
  - Inside __init__.py, database_setup.py, and daysnmeals.py, change
    ```
      engine = create_engine('sqlite:///whatsfordinner.db')
    ```
    to
    ```
      engine = create_engine('postgresql://catalog:password@localhost/catalog')
    ```
  - Create database schema: ```sudo python database_setup.py```
  - Populate and add data to database: ```sudo python populateCatalog.py```

## Create a new client_secrets.json file (Google OAuth)

  * Visit https://www.hcidata.info/host2ip.cgi
  * Input the IP address and get host name (http://ec2-34.218.78.235.compute-1.amazonaws.com/).
  * Go to the Google API Console.
  * From the left menu, select "API and Services", then "Credentials".
  * Click on the button "Create credentials", then select OAuth client ID" from the dropdown menu.
  * Add the Amazon Lightsail instance's public IP and the host name just found as Authorized JavaScript origins, then save.
  * Download the JSON file.
  * Open it and copy its content.
  * On the terminal, while logged as grader, cd into /var/www/WFD/WFD/.
  * Create a new JSON file with the command touch client_secrets.json.
  * Open the new file with nano client_secrets.json, and paste the previously copied content, then save the file.

## Install App dependencies:

  - sudo pip install Flask
  - sudo pip install sqlalchemy
  - sudo pip install Flask-SQLAlchemy
  - sudo pip install psycopg2
  - sudo apt-get install python-psycopg2
  - sudo pip install flask-seasurf
  - sudo pip install oauth2client
  - sudo pip install httplib2
  - sudo pip install requests



## Install virtual environment:

  - Install the virtual environment: ```sudo pip install virtualenv```
  - Create a new virtual environment: ```sudo virtualenv venv```
  - Activate the virutal environment: ```source venv/bin/activate```
  - Change permissions: ```sudo chmod -R 777 venv```


## Configure and Enable a New Virtual Host

  - Create WFD.conf file:
    ```
      sudo nano /etc/apache2/sites-available/WFD.conf

      <VirtualHost *:80>
	       ServerName 34.218.78.235
	       ServerAdmin cmudrenk@asu.edu
	       WSGIScriptAlias / /var/www/WFD/WFD.wsgi
	       <Directory /var/www/WFD/WFD/>
		         Order allow,deny
		         Allow from all
	       </Directory>
	       Alias /static /var/www/WFD/WFD/static
	       <Directory /var/www/WFD/WFD/static/>
		        Order allow,deny
		        Allow from all
	       </Directory>
	       ErrorLog ${APACHE_LOG_DIR}/error.log
	       LogLevel warn
	       CustomLog ${APACHE_LOG_DIR}/access.log combined
      </VirtualHost>
    ```
  - Enable the virtual host: ```sudo a2ensite WFD```


## Create .wsgi file:

    ```
      cd /var/www/WFD
      sudo nano WFD.wsgi
    ```
    - In catalogApp.wsgi file:
    ```
      #!/usr/bin/python
      import sys
      import logging
      logging.basicConfig(stream=sys.stderr)
      sys.path.insert(0,"/var/www/WFD/")

      from WFD import app as application
      application.secret_key = 'super_secret_key'
    ```

## Restart Apache and application is up and running:

    ```
      sudo service apache2 restart
    ```
    - Application is up and running in http://34.218.78.235


