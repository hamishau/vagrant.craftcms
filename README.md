# Introduction
This is a guide to setup a new installation of Craft CMS, running in an Ubuntu 22.04 Vagrant Box, on an Ubuntu 22.04 Host machine. There are probably better alternatives (I have used the official Craft CMS Docker images) but for me personally this is the easiest and cleanest way to get up and running locally with CraftCMS.

## Clone Repository

Clone Github repository to your local development machine:  
`git clone git@github.com:hamishcomau/vagrant-craftcms.git`

## Project Name

Search for all instances of `yourapp` and replace with your preferred project name.

## Deploy Vagrant Box

This requires both Virtualbox and Vagrant to be installed on your local machine:

* https://www.virtualbox.org/wiki/Downloads
* https://www.vagrantup.com/downloads.html

Once installed, we can simply run the Vagrant up command to setup the virtual machine and run the automated deployment script:

* `vagrant up`

Edit your local hosts file (not the hosts file on the Vagrant virtual machine) and add the following line:

* `192.168.56.10 yourapp.local`

The deployment process may take a bit of time to download the Vagrant box and step through the shell commands, once complete we can access the virtual machine:

* `vagrant ssh`

## MySQL Configuration

Create MySQL User and Database:

* `sudo mysql`
* `CREATE DATABASE yourapp;`
* `CREATE USER 'yourapp'@'%' IDENTIFIED BY '6b$uFrdR79FGkxY^';`
* `GRANT ALL PRIVILEGES ON yourapp.* TO 'yourapp'@'%';`
* `exit`

## Apache2 Configuration

Generate private root key and certificate:

* `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout yourapp.key -out yourapp.crt`

Enable SSL Apache2 module:

* `sudo a2enmod ssl`

Restart Apache2:

* `sudo systemctl restart apache2`

Update Virtual Host File by replacing the contents with the block below:

* `sudo nano /etc/apache2/sites-available/000-default.conf`

```
<VirtualHost *:443>
    ServerAdmin webmaster@localhost
    ServerName yourapp.local
    DocumentRoot /var/www/html/craft/web
    SSLEngine on
    SSLCertificateFile /home/vagrant/yourapp.crt
    SSLCertificateKeyFile /home/vagrant/yourapp.key
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    <Directory "/var/www/html/craft/web">
        allow from all
        Options -Indexes
        AllowOverride All
    </Directory>
</VirtualHost>

<VirtualHost *:80>
    ServerName yourapp.local
    Redirect / https://yourapp.local/
</VirtualHost>
```

Restart Apache2:

* `sudo systemctl restart apache2`

## Install Craft and Dependencies

Install Craft CMS, answer yes to all questions:

* `sudo composer create-project craftcms/craft /var/www/html/craft`

Which database driver are you using? `mysql`  
Database server name or IP address: `127.0.0.1`  
Database port: `3306`  
Database username: `yourapp`  
Database password: `6b$uFrdR79FGkxY^`  
Database name: `yourapp`

## Post Installation

These are not requirements, just helpful tools that might be needed such as importing a database or fixing up file/folder permissions.

Set all directories to 755 and files to 644, and set ownership to www-data:

* `cd /var/www/html`
* `find ./html -type d -exec chmod 0755 {} \;`
* `find ./html -type f -exec chmod 0644 {} \;`
* `sudo chown -R www-data:www-data ./craft`

Craft needs to have write access to:

* `sudo chmod 777 /var/www/html/craft/.env`
* `sudo chmod 777 /var/www/html/craft/composer.json`
* `sudo chmod 777 /var/www/html/craft/composer.lock`
* `sudo chmod 777 /var/www/html/craft/config/license.key`
* `sudo chmod -R 777 /var/www/html/craft/config`
* `sudo chmod -R 777 /var/www/html/craft/storage`
* `sudo chmod -R 777 /var/www/html/craft/vendor`
* `sudo chmod -R 777 /var/www/html/craft/web/cpresources`

Export MySQL database in Vagrant box:

* `sudo -s`
* `cd /var/www/html`
* `mysqldump yourapp > yourapp.sql`

Import MySQL database dump:

* `sudo -s`
* `cd /var/www/html`
* `mysql yourapp < yourapp.sql`
