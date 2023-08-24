# Introduction
This is a guide to setup a new installation of Craft CMS and Postgres, running in an Ubuntu 22.04 Vagrant Box. Install Vagrant and Virtualbox before proceeding.

## Clone Repository

Clone Github repository to your local development machine:
```
git clone git@github.com:hamishau/vagrant.craftcms.git
```

## Project Name

Search for all instances of `yourapp` and replace with your preferred project name.

## Deploy Vagrant Box

```
vagrant up
```

## Edit Hosts

Windows Powershell: `notepad c:\Windows\System32\Drivers\etc\hosts`  
Ubuntu Terminal: `sudo nano /etc/hosts`

```
192.168.56.10 yourapp.local
```

## Install Craft and Dependencies

Install Craft CMS, answer yes to all questions:

```
vagrant ssh -c 'sudo composer create-project craftcms/craft /var/www/html/craft'
```

Which database driver are you using? `pgsql`  
Database server name or IP address: `127.0.0.1`  
Database port: `5432`  
Database username: `postgres`  
Database password: `admin`  
Database name: `yourapp`

## Restart Apache2

```
vagrant ssh -c 'sudo systemctl restart apache2'
```

## Preview in browser:

```
http://yourapp.local:8080
```

---

### Post Installation
These are not required as a part of the Vagrant deployment process - just a collection of helpful commands that might be required at some point in your Vagrant box.

#### Set all directories to 755 and files to 644, and ownership to www-data:

```
vagrant ssh -c 'find /var/www/html -type d -exec chmod 0755 {} \; && find /var/www/html -type f -exec chmod 0644 {} \; && sudo chown -R www-data:www-data /var/www/html/craft'
```

#### Craft Folder Permissions:

```
vagrant ssh -c 'sudo chmod 777 /var/www/html/craft/.env && sudo chmod 777 /var/www/html/craft/composer.json && sudo chmod 777 /var/www/html/craft/composer.lock && sudo chmod 777 /var/www/html/craft/config/license.key && sudo chmod -R 777 /var/www/html/craft/config && sudo chmod -R 777 /var/www/html/craft/storage && sudo chmod -R 777 /var/www/html/craft/vendor && sudo chmod -R 777 /var/www/html/craft/web/cpresources'
```

#### Import PostgreSQL Database
```
vagrant ssh -c 'cd /var/www/html && sudo -u postgres psql yourapp < yourapp.psql'
```

#### Export PostgreSQL Database
```
vagrant ssh -c 'cd /var/www/html && sudo -u postgres pg_dump yourapp > yourapp.psql'
```

#### Drop PostgreSQL Database and Create New Database
```
vagrant ssh -c 'cd /var/www/html && sudo -u postgres dropdb yourapp && sudo -u postgres createdb yourapp'
```

#### Restart Vagrant Box
```
vagrant reload
```

#### Destroy Vagrant Box
```
vagrant destroy
```
