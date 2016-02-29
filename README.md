# AWS-Server-Configuration

This document describes how to setup a Linux server to host a Python web application using flask and PostgreSQL. 

## Summary
1. IP address: [52.36.251.6][1]
2. URL: [http://ec2-52-36-251-6.us-west-2.compute.amazonaws.com][2]
3. SSH port: 2200

## Installed sofwares

The following packages were installed sequencially. 

```
apt-get git
apt-get apache2
apt-get libapache2-mod-wsgi
apt-get postgresql
apt-get git
apt-get python pip
```
We also need `virtualenv` installed. 
```
pip install virtualenv
```

## Firewall settings
The `ufw` is enabled, and we only allow ports HTTP (80), NTP (123), and SSH (2200). 
```
ufw default deny incoming
ufw default allow outgoing
ufw allow 80
ufw allow 123
ufw allow 2200
ufw enable
```

## PostgreSQL settings

To create the user `catalog`, execute the following on the terminal

```
sudo -u postgres createuser -P -d catalog
```

To create the database `db_catalog` owned by `catalog`, execute the fllowing on the terminal

```
sudo -u postgres createdb -O catalog db_catalog
```

To enter the psql shell for the database `db_catalog`, execute the following 

```
psql -h localhost -U catalog -d db_catalog
```

To get help for psql, type `help` on the psql shell. 

## Setup Apache server
Create the directory `/var/www/catalog/` and then the file `/var/www/catalog/entry.wsgi` with the following content
```python
def application(environ, start_response):
  status = '200 OK'
  output = 'Hello World!'

  response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
  start_response(status, response_headers)
  
  return [output]
```
Create `catalog.conf` in `/etc/apache2/sites-available/` with the following content.
```xml
<VirtualHost *:80>
  WSGIScriptAlias / /var/www/catalog/entry.wsgi
  <Directory /var/www/catalog/src/>
    Order allow,deny
    Allow from all
  </Directory>
  Alias /static /var/www/catalog/src/static
  <Directory /var/www/catalog/src/static/>
    Order allow,deny
    Allow from all
  </Directory>
  ErrorLog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~
```

Deactivate the default website and enable the new website with the following commands.
```
a2dissite 000-default
a2ensite catalog
```
Restart Apache with the following command to apply the changes:
```
service apache2 restart
```

## Install the required Python modules

Change the directory to `/var/www/catalog/`.
```
cd /var/www/catalog
```
From this directory, we create an temporary Python environment using:
```
virtualenv venv # (venv is the name of the user's choice)
```
Now, we active the virtual environment with the following command:
```
source venv/bin/activate
```
From here, we install the following python packages:
```
pip install flask
pip install sqlalchemy
pip install psycopg2
pip install oauth2client
pip install httplib2
```
To deactivate the virtual environment `venv`, using the following command
```
deactivate
```

## Setup the server code
We first fetch the source of `Item-Catalog` from the github using:
```
git clone https://github.com/emguy/Item-Catalog.git src/
```
Replace the entire content of the file `/var/www/catalog/entry.wsgi` with the following
```python
#!/usr/bin/python
activate_this = "/var/www/catalog/venv/bin/activate_this.py"
execfile(activate_this, dict(__file__=activate_this))

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")

from src.web_server import app as application
application.secret_key = "Add your secret key"
application.debug = True
```
Add the following two URLs to the respective application in the google developers console
```
http://ec2-52-36-251-6.us-west-2.compute.amazonaws.com
```
```
http://52.36.251.6
```
Restart Apache with the following command to apply new changes.
```
service apache2 restart
```

[1]:http://52.36.251.6
[2]:http://ec2-52-36-251-6.us-west-2.compute.amazonaws.com


