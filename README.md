# Udacity - linux server configuration
This project learn how to configure linux server (aws) and deploy web app (python - Flask ) using vps amazon light sail . 

## Deployed version 
- URL : ``http://18.197.35.136/``
## Login to vps 
- ``$ ssh -i rsa grader@18.197.35.136 -p 2200``
- grader passwrod is : ``123456``
## Server configuration steps 
- #### Create amazon light sail instance
- #### Download ssh key from aws profile
- #### Connect to remote machine 
    - ``$ ssh -i LightsailDefaultPrivateKey.pem ubuntu@18.197.35.136``
- #### Update and upgrade OS 
    - `` $ sudo apt-get update `` , ``$ sudo apt-get upgrade`` 
- #### Add new user 
   - ``$ sudo adduser grader``
- #### change new user role
    - ``$ sudo visudo``
    - Add `` grader ALL=(ALL:ALL) ALL `` to the file and save
    - Login to new user ``$ su - grader``
- #### RSA
   -  __From local machine__ : ``$ ssh-keygen``  name file rsa this command generate two files rsa.pub (public key) and rsa (private key)
   -  Copy rsa (private key)
   -  __From remote machine__ : login to grader ``$ su - grader``
   -  ``$ sudo mkdir .ssh`` create .ssh folder 
   -  ``$ touch .ssh/authorized_keys`` create authorized_keys file
   -  ``$ sudo nano authorized_keys`` then paste rsa (private key) to this file
   -  ``$ chmod 700 .ssh`` making the folder readable, writable and executable by everyone
   -  ``$ chmod 644 .ssh/authorized_keys`` making file are readable and writeable by the owner of the file and readable by users in the group
   - ``$ sudo nano /etc/ssh/sshd_config`` , then change ``PasswordAuthentication`` to ``no``
   -  `$  sudo service ssh restart`
- #### Fire wall 
   - open network tab at account profile 
   - add another port 2200 aith portocol tcp 
   - ``$ sudo ufw allow 2200/tcp``
   - ``$ sudo ufw allow 123/udp``
   - ``$ sudo ufw allow ntp``
   - ``$ sudo ufw enable``

- #### Time zone
  - ``$ sudo dpkg-reconfigure tzdata``
  - select none of above
  - select utc 
- #### Apache setup 
   - ``$ sudo apt-get install apache2``
   - ``$ sudo apt-get install libapache2-mod-wsgi``
   - ``$ sudo service apache2 restart``
   - open ``http://18.197.35.136/`` you will see apache default page
- #### Postgres configration 
  - ``$ sudo apt-get install postgresql``
  - ``$ sudo su - postgres``
  - ``$ psql``
  - ``$ CREATE USER kareem WITH PASSWORD '123456';``
  - ``$ CREATE DATABASE itemsCatalog``
  - ``$ GRANT ALL PRIVILEGES ON DATABASE itemsCatalog TO kareem``
- #### Git - clone repository
   - ``$ sudo apt-get install git``
   - __From local machine :__ 
      - change the ``app.py`` to `` __init__.py``
      - ``create_engine('postgresql://kareem:myPassword@localhost/itemsCatalog')``
       set at ``__init__.py`` and ``database_setup.py`` instead of sqlite enginge 
      - ``$ git push origin master`` update remote repository 
   - ``$ cd /var/www`` , ``$ sudo mkdir ItemCatalog`` 
   - ``$ sudo chmod -R 777 ItemCatalog``, ``$ cd ItemCatalog``
   - ``$ git clone https://github.com/KareemMohamed0/ItemCatalog.git``
   - ``$ sudo apt-get install python-pip``
   - ``$ sudo pip install -r requirements.txt``
   - ``$ sudo python database_setup.py``
- #### Run the app 
  - `$ sudo nano /etc/apache2/sites-available/Website.conf` insert next code snipet in this file :
    ```xml
    <VirtualHost *:80>
            ServerName 18.197.35.136
            ServerAdmin grader
            WSGIScriptAlias / /var/www/ItemCatalog/ItemCatalog.wsgi
            <Directory /var/www/ItemCatalog/ItemCatalog/>
                    Order allow,deny
                    Allow from all
            </Directory>
            Alias /static /var/www/ItemCatalog/ItemCatalog/templates
            <Directory /var/www/ItemCatalog/ItemCatalog/templates/>
                    Order allow,deny
                    Allow from all
            </Directory>
            ErrorLog ${APACHE_LOG_DIR}/error.log
            LogLevel warn
            CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
  -   ``$ cd /var/www/ItemCatalog``
  -   ``$ sudo nano ItemCatalog.wsgi`` insert next code snipet in this file :
  
        ```python
        #!/usr/bin/python
        import sys
        import logging

        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/ItemCatalog/")
        
        from ItemCatalog import app as application
        application.secret_key = 'dsndjdnsjdns'
        ```
    - ``$ sudo service apache2 restart``
    - open ``http://18.197.35.136/`` my website is deployed successfully
- #### Update packages issue 
    - ``$ sudo apt-get update && sudo apt-get dist-upgrade``
- #### Refrences
    - [disabling-password-authentication-on-your-server](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server#disabling-password-authentication-on-your-server)
    - [how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-16-04)
    - [ubuntu-server-message-says-packages-can-be-updated-but-apt-get-does-not-update](https://serverfault.com/questions/265410/ubuntu-server-message-says-packages-can-be-updated-but-apt-get-does-not-update)
    - [how-to-deploy-a-flask-application-on-an-ubuntu-vps] (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
