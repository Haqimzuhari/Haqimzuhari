
# Host for production a Laravel project on Linux(ubuntu) VPS from Github

#### Laravel project that using Livewire, unable to be run in Shared Hosting. Thus, VPS is the only solution to hosting a Laravel project for production.  Linux VPS Ubuntu distro is the easiest one to setup to<br><br>VPS can be subscribe from [Digital Ocean](https://www.digitalocean.com/). One of the best so far. Very flexible and affordable. This is not paid review. Just my two cents.<br><br>But if Laravel project did't not use Livewire(why not using Livewire???) then Shared Hosting is good to go.

## First of all
##### Server environment need to be setup first.<br>Please follow [this](https://github.com/Haqimzuhari/Haqimzuhari/blob/master/ubuntu-lamp-stack-terminal.md) instruction for setting up server environment

## Install and setup Laravel dependencies
### Install Node and NPM
Installing Node.js from it's official [Github](https://github.com/nodesource/distributions/blob/master/README.md) page. Recommended to install latest version of Node.js<br>NPM will automatically installed when Node.js is installed
```bash
# Check Nodejs availibility & version
node -v

# Check NPM availibility & version
npm -v
```
### Install Composer
This installation step is from official [Composer](https://getcomposer.org/download/) page. Recommended to install latest version of Composer
```bash
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '756890a4488ce9024fc62c56153228907f1545c228516cbf63f885e036d37e9a59d27d63f46af1d4d07ee0f76181c7d3') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"

# To allow Composer to be use globally
sudo mv composer.phar /usr/local/bin/composer
```
```bash
# Check Composer availibility & version
composer -v
```
### Install Laravel
This installation step is from official [Laravel](https://laravel.com/docs/8.x#installation-via-composer) page via Composer. Recommended to install latest version of Laravel
```bash
composer global require laravel/installer

# Add Laravel into global path
echo 'export PATH="$PATH:$HOME/.config/composer/vendor/bin"' >> ~/.bashrc
source ~/.bashrc
```
```bash
# Check Laravel availibility & version
laravel -v
```
### Install Git
This installation step is from official [Git](https://git-scm.com/download/linux) page for Linux. Recommended to install latest version of Git
```bash
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git
```
```bash
# Check Git availibility & version
git --version
```
## Setup Database
### Create a database user other than root
It's is recommended to not use root user in production application
```bash
# Login into mysql as root
mysql -u root -p
<password>
```
```sql
# Create new user
CREATE USER 'user'@'localhost' IDENTIFIED BY 'password';
GRANT ALL PRIVILEGES ON * . * TO 'user'@'localhost';
FLUSH PRIVILEGES;
exit
```
```bash
# Login into mysql as new user
mysql -u root -p
<password>
```
```sql
# Create Database
CREATE DATABASE database_name;
exit
```
## Cloning project from Github
### Setup SSH between VPS and Github for security
 As this MD file is written at 2021, Github has update their policy where user are unable to close, push or pull only by using Username and Password. Need to use SSH or probably Access Token.
 
### Generate SSH from VPS Server
```bash
# Generate new pair of SSH key
ssh-keygen

Generating public/private rsa key pair.
Enter file in which to save the key (/home/{your-user-name}/.ssh/id_rsa):
```
Hit enter to use default value `/home/{user}/.ssh/id_rsa` or you can key-in your preferred name such as `/home/{user}/.ssh/id_github` and you can find the SSH file in `/home/{user}/.ssh` directory
```bash
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
Hit enter to **not use** passpharse. Passphare is to add one more security layer on SSH key if the PC use to access server not only be use by you. Else, you can just skip this passphrase.
```bash
The key fingerprint is:
SHA256:{sha256-key} {user}@{machine}
The key's randomart image is:
+---[RSA 3072]----+
|Xoo.= .o.  ..    |
|=@oo * .....     |
|@*+ o . Eo=      |
|X=o  o o.* .     |
|o+o.o o S o      |
|. .. o . .       |
|        .        |
|                 |
|                 |
+----[SHA256]-----+
```
This indicate a new pair of SSH key has been generated. You can find the SSH file in `/home/{user}/.ssh` directory
### Copy SSH key
SSH key will be generate 2 files in `/home/{user}/.ssh`
```bash
{your-ssh-key-name}
and
{your-ssh-key-name}.pub
```
Open .pub SSH key and copy
```bash
cat ~/.ssh/{your-ssh-key-name}.pub
```
SSH key will display and simply copy the SSH key to clipboard
### Add SSH key to your Github
1. Open your [Github Setting SSH and GPG Keys](https://github.com/settings/keys)
2. Click `New SSH Key` to add new SSH key
3.  Add title of your SSH key such as `Home-computer`
4. Paste your .pub SSH key into `key` then click `Add SSH Key` to save
### Cloning Laravel Project
1. Choose your repository
2. Click `Code` button to get link to clone. Copy `SSH` link of your repository
3. Open VPS terminal
4. Run `git clone` command
```bash
git clone {SSH-link} /var/www/example.com
```
## Setup Laravel Environment
### As usual
```bash
# Hop into project directory
cd /var/www/example.com

# Composer
composer install

# NPM
sudo npm install
npm run dev

# Copy .env file
cp .env.example .env

# Modify .env file
sudo nano .env

# Artisan command
php artisan key:generate

# Migrate
php artisan migrate
# Or with seed
php artisan migrate --seed
```

##### That pretty much it.