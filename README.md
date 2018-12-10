
# udacity-linux-server-configuration

### Description
To install a linux on virtual machine and make it host your web applications.

- IP address: 35.234.117.82
- Accessible SSH port: 2200



### Methods used

1. Create new user named grader and give it the permission to sudo
  - login to the server with ssh ` ssh -i ~/.ssh/root_key root@35.234.117.82`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add this`grader ALL=(ALL:ALL) NOPASSWD:ALL`
  - Run `sudo nano /etc/hosts`
  - To solve this error`sudo: unable to resolve host` add this line `127.0.1.1 ip-10-20-52-12`

2. Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - then run `sudo service ssh restart`
  - Confirm it

4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`

5. Change local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata`  and then none of the above and will show UTC now choose it

6. Configure key-based authentication for grader user
  - Run this command ` sudo mkdir /home/grader/.ssh`
then run this
` sudo nano /home/grader/.ssh/authorized_keys`
and paste your public key
7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@35.234.117.82`

8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`


10. Clone from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www`
  - `sudo mkdir catalog`
  - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
  - `cd /catalog`
  - Clone your project from github `git clone https://github.com/FahadAlsubaie/Item-catalog.git catalog`
  - Create a catalog.wsgi file, with this content
  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'
  ```
  - Rename application.py to __init__.py `mv application.py __init__.py`

11. Install virtual environment
  - First install pip with this command
`sudo apt-get install python-pip`
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate it with
  - `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

13. Install Flask and other dependencies
  - Install Flask `pip install Flask`
  - Install all others project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

14. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`

15. Configure a new virtual host
  - Create a new file with this : `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Put this code:
  ```
  <VirtualHost *:80>
      ServerName 35.234.117.82
      ServerAlias
      ServerAdmin admin@35.234.117.82
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
  - Enable the virtual host `sudo a2ensite catalog`

16. Install and setup PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - now you need to change  the create engine line in your `__init__.py` and `database_setup.py` to:
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - `python /var/www/catalog/catalog/database_setup.py`
17. Restart Apache
  - `sudo service apache2 restart`

18. Visit site at [http://35.234.117.82/](http://35.234.117.82/)

**I'd like to thanks *[rrjoson](https://github.com/rrjoson)* for his  useful README it helped me with this project**
