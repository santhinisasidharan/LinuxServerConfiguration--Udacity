# LinuxServerConfiguration--Udacity
This is done as a part of 5th project in Udacity Full stack developer course.Here you will take a baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.
We are deploying Item catalog app from project 3.

## Project Info.
IP ADDRESS:13.126.215.216
SSH PORT: 2200
LOGIN LINK :[Click here](http://13.126.215.216)
## Get your server-
1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
- Log in! - First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
- Create an instance - Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter.
- Choose an instance image: Ubuntu- Lightsail supports a lot of different instance types. An instance image is a particular software setup, including an operating system and optionally built-in applications. For this project, you'll want a plain Ubuntu Linux image. There are two settings to make here. First, choose "OS Only" (rather than "Apps + OS"). Second, choose Ubuntu as the operating system.
- Choose your instance plan - The instance plan controls how powerful of a server you get. It also controls how much money they want to charge you. For this project, the lowest tier of instance is just fine. And as long as you complete the project within a month and shut your instance down, the price will be zero.
- Once your instance has started up, you can log into it with SSH from your browser.The public IP address of the instance is displayed along with its name. When you SSH in, you'll be logged as the ubuntu user. When you want to execute commands as root, you'll need to use the sudo command to do it. 

## Secure your server
### 1.Create a user named grader and give sudo permissions
1. Log into the remote VM as root user .
2. Add a new user called grader.
3. Create a new file under the suoders directory . Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.  
`$ ssh root@13.126.215.216`
`$ sudo adduser grader`
`$ sudo nano /etc/sudoers.d/grader`
## Update all installed packages
`$ sudo apt-get update`
`$ sudo apt-get upgrade`
## key based authentication enabled for grader user
1. Generate key on your local machine. with: `$ ssh-keygen -f ~/.ssh/project_key.rsa`.
2. Log into the remote VM as root user through ssh and create file: `$ touch /home/grader/.ssh/authorized_keys`.
3. Copy the content of the udacity_key.pub file from your local machine to the /home/grader/.ssh/authorized_keys file you just created on the remote VM. Then change some permissions:
`$ sudo chmod 700 /home/grader/.ssh`.
`$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
Make grader root user: `$ sudo chown -R grader:grader /home/grader/.ssh`.
4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh grader@13.126.215.216 -i ~/.ssh/project_key.rsa`.
5. To enforce key based authentication `$ sudo nano /etc/ssh/sshd_config`. Find the *PasswordAuthentication* line and edit it to *no*.Then `$ sudo service ssh restart`.

## Change SSH port from 22 to 2200
`$ sudo nano /etc/ssh/sshd_config`. 
Find the *Port* line and edit it to *2200*.
`$ sudo service ssh restart`.
Log into the remote VM through ssh with the following command: `$ ssh grader@13.126.215.216 -i ~/.ssh/project_key.rsa -p 2200`.

## Disable ssh login for root user
`$ sudo nano /etc/ssh/sshd_config`. 
Find the *PermitRootLogin* line and edit it to *no*.
`$ sudo service ssh restart`.

## Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
`$ sudo ufw allow 2200/tcp`.
`$ sudo ufw allow 80/tcp`.
`$ sudo ufw allow 123/udp`.
`$ sudo ufw enable`.

## Install Apache, mod_wsgi- Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications.
`$ sudo apt-get install apache2`.
`$ sudo apt-get install libapache2-mod-wsgi python-dev`.
Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
`$ sudo service apache2 start`.

## Install git - configure username and mail
`$ sudo apt-get install git`.
`$ git config --global user.name <username>`.
`$ git config --global user.email <email>`.
To clone the Item Catalog app from github:
`$ cd /var/www`. Then: `$ sudo mkdir catalog`.
Make grader owner for the catalog folder: `$ sudo chown -R grader:grader catalog`.
Move inside that newly created folder: `$ cd /catalog` and clone the catalog repository from Github: `$ git clone https://github.com/santhinisasidharan/P5-Item-Catalog.git catalog`
 Make a *catalog.wsgi* file to serve the application over the *mod_wsgi*. That file should look like this:
 ```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
 To deploy the catalog app, so I made a *deployment* branch which slightly differs from the *master*. Move inside the repository, `$ cd /var/www/catalog/catalog` and change branch with: `$ git checkout deployment`.

## Install virtual environment and flask
Install virtualenv `$ sudo pip install virtualenv`.
Move to the *catalog* folder: `$ cd /var/www/catalog`. Then create a new virtual environment with the following command: `$ sudo virtualenv venv`.
Activate the virtual environment: `$ source venv/bin/activate`.
Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.
Install Flask: `$ pip install Flask`.
Install all the other project's dependencies: `$ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`.

## Configure and enable virtual host

Create a virtual host config file and paste in the following lines of code:: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
```
<VirtualHost *:80>
    ServerName 13.126.215.216
    ServerAlias 
    ServerAdmin admin@13.126.215.216
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
Enable the new virtual host: `$ sudo a2ensite catalog`.

## Install and configure PostgreSQL
`$ sudo apt-get install libpq-dev python-dev`.
`$ sudo apt-get install postgresql postgresql-contrib`.
Postgres automatically creates a new user during its installation, whose name is 'postgres'.Let's change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
Create a new user called 'catalog' with password: `# CREATE USER catalog WITH PASSWORD 'hello123';`
Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.
Connect to the database: `# \c catalog`.
Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:hello123@localhost/catalog')
```
Setup the database with: `$ python /var/www/catalog/catalog/database_setup.py`.

## Update OAuth authorized JavaScript origins
Add http://13.126.215.216

## Restart Apache to launch the app
`$ sudo service apache2 restart`.

