# Linux-Server
- Project 5- FSND

### Project link: [link](http://ec2-13-126-131-204.ap-south-1.compute.amazonaws.com/)
### In this project we had to deploy item catalog project on linux server.
### Project Details:
* public Ip: `13.126.131.204`
* SSH PORT: `2200`
* Full project URL:[link](http://ec2-13-126-131-204.ap-south-1.compute.amazonaws.com/)



### Commands:

* Initialization :
    * create an account on amazon lightsail.
    * log in to your account.
    * create an ubuntu instance on amazon light sail. URL: [link](https://lightsail.aws.amazon.com/ls/webapp/home)
    * click on the instance and download default ssh key.
    * open terminal by clicking on `Connect using SSH` button
    
* Amazon Lightsail terminal :
    * Copy your public IP address written on bottom left of terminal.
    * Copy default key to /.ssh folder whith help of nano text editor by typing `ctrl+o`, enter and `ctrl+x`, let's say we named it 'key'.
    * give command `chmod 600 ~/.ssh/key`
    * Type `ssh -i ~/.ssh/key ubuntu@*YOUR_PUBLIC_IP_ADDRESS_HERE*` to create and authenticate the instance.

* Creating user - 'grader' :
    * `sudo adduser grader`
    * `apt-get install finger`
    * `finger grader`


* Give the grader the permission to sudo :
    * type `sudo visudo`
    * In opened file below '#User privilege specification' add `grader   ALL=(ALL:ALL) ALL` below the root user and save the file.
    * Add grader to `/etc/sudoers.d/`.Type command `sudo nano /etc/sudoers.d/grader` and add `grader   ALL=(ALL:ALL) ALL` to file.
    * Add root to `/etc/sudoers.d/`.Type command `sudo nano /etc/sudoers.d/root`and add `root   ALL=(ALL:ALL) ALL` to file.


* Updating all packages :
    * `sudo apt-get update`
    * `sudo sudo apt-get upgrade` Press Y for yes.

* Change the SSH port from 22 to 2200 and configuring SSH :
    *  Type in `nano /etc/ssh/sshd_config` add `Port 2200` below `Port 22` in file.
    * Change `PermitRootLogin prohibit-password` to `PermitRootLogin no` to revoke root login
    * Change `PasswordAuthentication` from `no` to `yes`.
    * save file by pressing `ctrl+o`, enter, `ctrl+x`.
    * restart ssh service`sudo service ssh reload` and `sudo service ssh restart`.


* Generating SSH key :
    * Type `ssh-keygen` and give path to save example: `/home/ubuntu/.ssh/item-catalog`.
    * Enter pass Phrase or leave it blank.
    * open machine's networking tab on lightsail and add custom TCP application with 2200 port.
    
* Saving SSH key to grader account :
    * Type `ssh -v grader@*Public-IP-Address* -p 2200` to log in with grader.
    * Type `mkdir .ssh` to make a directory named .ssh
    * Create file by giving command `touch .ssh/authorized_keys` to store generated key.
    * Read contents of key by command `cat /home/ubuntu/.ssh/item-catalog` or your path and copy the key.
    * Paste the key in the file you just created in grader `nano .ssh/authorized_keys` and save file.
    * Type in `chmod 700 .ssh`, `chmod 644 .ssh/authorized_keys` to give permissions.
    * type `nano /etc/ssh/sshd_config` and change `PasswordAuthentication` from `yes` to `no`. Save file.  
    * login with key pair: `ssh grader@Public-IP-Address* -p 2200 -i /home/ubuntu/.ssh/item-catalog`


* Configuring the UFW :
    * Check UFW statu : `sudo ufw status`
    * `sudo ufw default deny incoming`
    * `sudo ufw default allow outgoing`
    * `sudo ufw allow ssh`
    * `sudo ufw allow 2200/tcp`
    * `sudo ufw allow 80/tcp`
    * `sudo ufw allow 123/udp`
    * `sudo ufw enable`


* Configuring local timezone :
    * `sudo dpkg-reconfigure tzdata`. Select time zone.


* Installing and configuring Apache :
    * `sudo apt-get install apache2`.
    * open your public IP address in browser and check.
    * `sudo apt-get install libapache2-mod-wsgi` installing mod_wsgi.
    * `sudo nano /etc/apache2/sites-enabled/000-default.conf`.
    * In file add `WSGIScriptAlias / /var/www/html/myapp.wsgi` before `</VirtualHost>` and save file.
    * Restart Apache `sudo service apache2 restart`


* installing git and python :
    * `sudo apt-get install git`.
    * `sudo apt-get install python-dev`.
    * Verify wsgi is enabled `sudo a2enmod wsgi`
    
* Create a small flask app.
    * `cd /var/www` to enter www directory.
    * `sudo mkdir catalog` make directory catalog.
    * `cd catalog` to enter catalog.
    * `sudo mkdir catalog` make another directory catalog.
    * `cd catalog` to enter catalog.
    * `sudo mkdir static templates` to make directories static and templates to store static and templates of project.
    * `sudo nano __init__.py ` to create init file and add following code.

    ```
     from flask import Flask
    app = Flask(__name__)
    @app.route("/")
    def hello():
        return "Hello, world (Testing!)"
    if __name__ == "__main__":
    app.run()
    ```

* installing flask :
    * Type `sudo apt-get install python-pip` to install python-pip
    * Type `sudo -H pip install virtualenv ` to install virtualenv
    * Command `sudo virtualenv venv` to create obj of virtualenv
    * Command `sudo chmod -R 777 venv` to give permission.
    * Type `source venv/bin/activate`
    * Type `sudo -H pip install Flask` install Flask.
    * Type `sudo -H pip install flask` install flask.
    * Type `python __init__.py` run init file.
    * Type `deactivate`

* Configure And Enable New Virtual Host :
    * create file `sudo nano /etc/apache2/sites-available/catalog.conf` and paste following code:

    ```
    <VirtualHost *:80>
      ServerName 34.201.114.178
      ServerAdmin admin@34.201.114.178
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
    * save file.
    * `sudo a2ensite catalog`
    * `cd /var/www/catalog`
    * `sudo nano catalog.wsgi` and paste following code:

    ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
  ```

  * save file.
  * `sudo service apache2 restart` restarts apache and loads changes.

* Clone Github Repo containing project :
    * `sudo git clone https://github.com/vaishnavimehta/item-catalog`
    * `shopt -s dotglob`. Move files from clone directory to catalog directory `mv /var/www/catalog/item-catalog/*
    /var/www/catalog/catalog/`
    * `sudo rm -r devpost` to remove clone directory as it is useless now.

* redirecting .git :
    * `cd /var/www/catalog/`
    * `sudo nano .htaccess`. paste `RedirectMatch 404 /\.git` in file open and save file.

* install modules for project:
    * `source venv/bin/activate`
    * `sudo -H pip install httplib2`
    * `sudo -H pip install requests`
    * `sudo -H pip install --upgrade oauth2client`
    * `sudo -H pip install sqlalchemy`
    * `sudo -H pip install Flask-SQLAlchemy`
    * Id your project uses any other modules, install them.


* Installation and configuration of PostgreSQL :
     * Type `sudo apt-get install postgresql` to install postgresql.
    * Type in `sudo apt-get install postgresql-contrib`.
    * `sudo nano database_setup.py` to open db file.
    * `python engine = create_engine('postgresql://catalog:db-password@localhost/catalog')` to change sqlite command.
    * Make same changes in finalproject.py file.
    * `mv project.py __init__.py` copies finalproject.py file into __init__.py file.
    * `sudo adduser catalog` adds catalog user.
    * login as postgres super user`sudo su - postgres` and `psql` to enter postgres.
    * `CREATE USER catalog WITH PASSWORD 'db-password';`
    * ` ALTER USER catalog CREATEDB;`
    * `\du`
    * `CREATE DATABASE catalog WITH OWNER catalog;` create DB with catalog as owner.
    * `\c catalog` connect DB.
    * `REVOKE ALL ON SCHEMA public FROM public;` so that only catalog can use, view and modify it.
    * `GRANT ALL ON SCHEMA public TO catalog;`
    * `\q` to exit psql.
    * `exit` to exit postgresql.
    * `python database_setup.py` to set up project DB.



* login configuration
    * google cannot have an IP as origin so we create a hostname.
    * click [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi) and enter your public IP to get your host name.
    * `sudo nano /etc/apache2/sites-available/catalog.conf` and add `ServerAlias YOURHOSTNAME` under `ServerAdmin`.
    * enable virtual host`sudo a2ensite catalog`
    * `sudo service apache2 restart`
    * Open google developer console and add your host name and IP address to Authorized Javascript origins.
    * Add *YOURHOSTNAME*/login to the Authorized redirect URIs.
