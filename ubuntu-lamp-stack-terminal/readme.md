# LAMP Stack using Terminal on Ubuntu
#### Setup Linux, Apache, Mysql, PHP using terminal on Ubuntu

## Update & Upgrade
```bash
sudo apt update
sudo apt upgrade
```

## Apache, Mysql, PHP Installation
#### Installing Apache2
```bash
sudo apt install apache2
```
#### Installing Curl
```bash
sudo apt install curl
```
#### Installing Mysql Server
```bash
sudo apt install mysql-server
sudo mysql_secure_installation

# Setup
Validate Password:Y
Password Strength:2
Change root password/use this password:Y
Y to all others setup

# Log into mysql as sudo and set root password to use natively
sudo mysql
```
``` sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
FLUSH PRIVILEGES;
exit
```
``` bash
# Now test login mysql using cli
mysql -u root -p

# Enter password when password being ask then hit enter
# Should enter into mysql cli then exit
exit
# This process just to make sure we can access mysql db for our application later
```
#### Installing PHP

```bash
# Add repository
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:ondrej/php

# For available latest version of PHP with extensions
sudo apt install php php-mysql php-mbstring php-fileinfo php-bcmath php-dom php-xml

# For specific version of PHP such as PHP 7.4 with extensions
sudo apt install php7.4 php7.4-mysql php7.4-mbstring php7.4-fileinfo php7.4-bcmath php7.4-dom php7.4-xml
```

## Environment Configuration
#### Set index.php as priority
```bash
sudo nano /etc/apache2/mods-enabled/dir.conf

# Make sure index.php on first order. Just sort
<IfModule mod_dir.c>
	DirectoryIndex index.php index.html index.cgi index.pl index.xhtml index.htm
</IfModule>
```
#### Setup Virtual Host
```bash
cd /var/www
sudo mkdir example.com
sudo chown -R $USER:$USER example.com
sudo chmod -R 777 example.com
cd /etc/apache2/site-available
sudo cp 000-default.conf example.com.conf
sudo nano example.com.conf

Change <VirtualHost *:80> port if needed
Add ServerName under ServerAdmin: ServerName example.com
Add ServerAlias under ServerName: ServerAlias www.example.com
DocumentRoot /var/www/example.com or /var/www/example.com/public for Laravel project
ctrl+x then y then enter to exit

sudo a2ensite domain.com.conf
sudo a2dissite 000-default.conf
sudo apache2ctl configtest
sudo systemctl restart apache2
```
#### Enable .htaccess
```bash
sudo a2enmod rewrite
sudo nano /etc/apache2/sites-available/example.com.conf


<VirtualHost>
	# Add this anywhere inside <VirtualHost></VirtualHost>
	<Directory /var/www/html>
		Options Indexes FollowSymLinks MultiViews
		AllowOverride All
		Require all granted
	</Directory>
</VirtualHost>

ctrl+x then y then enter to exit

sudo systemctl restart apache2
```
#### Check PHP Info
```bash
cd /var/www/example.com
sudo nano phpinfo.php
```
```php
<?php phpinfo() ?>
```
```bash
ctrl+x then y then enter to exit
```
Open browser and go to `http://example.com/phpinfo.php`
