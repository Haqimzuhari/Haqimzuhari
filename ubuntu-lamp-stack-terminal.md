
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
#### Installing Zip UnZip
```bash
sudo apt install zip unzip
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
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY  'password';
FLUSH PRIVILEGES;
exit
```

``` bash
# Now test login mysql using cli
mysql -u root -p
<password>

# Should enter into mysql cli then exit
# This process just to make sure we can access mysql db for our application later
exit
```
#### Installing PHP
```bash
# Add repository
sudo apt -y install software-properties-common
sudo add-apt-repository ppa:ondrej/php

# For available latest version of PHP with extensions
sudo apt install php php-mysql php-mbstring php-fileinfo php-bcmath php-dom php-xml php-zip

# For specific version of PHP such as PHP 8.0 with extensions
sudo apt install php8.0 php8.0-mysql php8.0-mbstring php8.0-fileinfo php8.0-bcmath php8.0-dom php8.0-xml php8.0-zip

# For specific version of PHP such as PHP 7.4 with extensions
sudo apt install php7.4 php7.4-mysql php7.4-mbstring php7.4-fileinfo php7.4-bcmath php7.4-dom php7.4-xml php7.4-zip
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
#### Check PHP Info
```bash
sudo nano /var/www/html/phpinfo.php
```
```php
<?php  phpinfo()  ?>
```
```bash
ctrl+x then y then enter to exit
```
Open browser and go to `http://example.com/phpinfo.php` or using your public ip `http://111.111.111.111/phpinfo.php`

After setup DNS domain name point to public ip, now application can be access using domain name

##### Don't forget to remove `phpinfo.php` once finish view on browser
#### How to hosting Laravel project on VPS Linux Server? Click [here](https://github.com/Haqimzuhari/Haqimzuhari/blob/master/host-laravel-on-linux-vps.md)
