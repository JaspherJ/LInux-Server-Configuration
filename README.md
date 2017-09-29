# Project-6 Linux Server Configuration
A baseline installation of a Linux server to host a Flask web application. It also includes securing server from attack vectors, install and configure a database server, and deploy it.
 
- IP Address: 18.221.181.198
- Port: 2200
# Configuration
**Creating a new user and giving sudo permissions**
1. New user creation <br />`sudo adduser grader`
2. Giving sudo permission  <br> `sudo nano /etc/sudoers.d/grader` <br>  copy this line<br>
`grader ALL=(ALL:ALL) ALL`

**Updating packages**
1. Run `sudo apt-get update` 
2. Run `sudo apt-get upgrade`

**Change SSH port from 22 to 2200**
1. Run sudo nano /etc/ssh/sshd_config
2. Locate the line with port number 22
3. Change the port from 22 to 2200
4. Also change the `PermitRootLogin yes ` `PermitRootLogin no`

**Forcing user to login with key based authentication**
1. Run `mkdir /home/grader/.ssh`
2. Chang the directory permission `chmod 700 /home/grader/.ssh`
3. Type `touch .ssh/authorized_keys`
4. Run `sudo nano .ssh/authorized_keys ` and copy the public key downloaded

**Configuration of Uncomplicated Firewall to allow SSH , HTTP and NTP ports**
1. Run `sudo ufw allow 2200/tcp`
2. Run `sudo ufw allow 80/tcp`
3. Run `sudo ufw allow 123/udp`
4. Run `sudo ufw enable`

**Changing the time zone**
* Run `sudo timedatectl set-timezone UTC`

**Installing Configuring Apache and mod_wsgi to host python web application**
1. Install Apache Web server</br>
`sudo apt-get install apache2`
2. Run `sudo apt-get install libapache2-mod-wsgi python-dev`
3. Activate mod_wsgi
 `sudo a2enmod wsgi`
4. Run `sudo service apache2 start` to start the server

**Installing Github and cloning the Catalog Application**
1. Run `sudo apt-get install git`
2. Move to `cd /var/www`
3. Make a new directory</br>
`sudo mkdir catalog`
4. Change the owner of the the directory </br>
`sudo chown -R grader:grader catalog`
5. Move to the directory  `cd /catalog` and clone using </br>
`git clone https://github.com/JaspherJ/Building-an-Item-Catalog.git`
6. Create a new file `sudo nano catalog.wsgi` and copy the contents </br>
```
 import sys
 import logging
 logging.basicConfig(stream=sys.stderr)
 sys.path.insert(0, "/var/www/catalog/")
 
 from catalog import app as application
 application.secret_key = 'super_secret_key' 
```
7. Also rename `project.py` to `__init.py__` inside the catalog folder
8. After renaming change the path of `clients_secret.json` file </br>
`/var/www/catalog/catalog/client_secrets.json`

Installing and configuring virtual environment
1. Run `sudo pip install virtualenv`
2. Run `sudo virtualenv venv`
3. Activate the virtual environment `source venv/bin/activate`
4. Change permissions  `sudo chmod -R 777 venv`
5. Run `sudo nano /etc/apache2/sites-available/catalog.conf`
6. Copy and paste this code
```
<VirtualHost *:80>
    ServerName 18.221.181.198
    ServerAdmin admin@18.221.181.198
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
7. Enable `sudo a2ensite catalog`

**Installing and Configuring PostgreSQL**
1. Run `sudo apt-get install libpq-dev python-dev`
2. Install postgres `sudo apt-get install postgresql postgresql-contrib`
3. Switch to postres user `sudo su - postgres`
4. Connect to the psql by `psql` in terminal
5. Creating a user `catalog` </br>
 ` CREATE USER catalog WITH PASSWORD 'catalog';`
6. And then give the following commands
```
CREATE DATABASE catalog WITH OWNER catalog;
\c catalog
REVOKE ALL ON SCHEMA public FROM public;
GRANT ALL ON SCHEMA public TO catalog
```
7. Exit PostgreSQL and postgres user </br>
Type `\q` and `$ exit`

8. Change the engine variable line in both `database_setup.py` and `__init__.py` <br>
```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
``` 
9. Run `python database_setup.py`
10. Restart apache `sudo service apache2 restart`

**Run the application from the web browser** <br>
 - [http://18.221.181.198](http://18.221.181.198)




### References and Source Code

- [Deploying to Linux server(Udacity)](https://classroom.udacity.com/nanodegrees/nd004/parts/ab002e9a-b26c-43a4-8460-dc4c4b11c379)
- [Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
- [Ubuntu documentation](https://help.ubuntu.com/community/UFW)
- [Setting up Flask](https://help.pythonanywhere.com/pages/Flask/)
- [Ask Ubuntu](https://askubuntu.com/)
