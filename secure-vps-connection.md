# Securing  Linux VPS server (Ubuntu)
### Generate SSH Private Key
Generate SSH private key by following [this](https://github.com/Haqimzuhari/Haqimzuhari/blob/master/generate-ssh-private-key.md) procedure
### Add Public SSH Key to VPS
Log into server
```bash
ssh root@{ip-address}
password:
```

> Change password from default password if needed

Paste copied SSH Public Key
```bash
sudo nano ~/.ssh/authorized_keys

ctrl-x then y then Enter to save and exit
```
Exit server
```bash
exit
```
### Login using SSH Private Key
Log into server using SSH Private Key
```bash
ssh -i ~/.ssh/{ssh-private-key-name} root@{ip-address}
```
### Create New User
```bash
adduser newuser

# Set Password
Enter new UNIX password:
Retype new UNIX password:

# Just skip other things

Is the information correct?: `Y`
```
### Grant New User as Sudoer
```bash
usermod -aG admin newuser
```
### Create `authorized_keys` file for New User
Switch current user to New User
```bash
sudo su newuser
```
Create `.ssh` folder and `authorized_keys` file and paste SSH Public Key
```bash
sudo nano ~/.ssh/authorized_keys

ctrl-x then y then Enter to save and exit
```
### Disable default connection method and disable connect using `root` 
Log into server as New User by using SSH Private Key
```bash
ssh -i ~/.ssh/{ssh-private-key-name} newuser@{ip-address}
```
Modify `sshd_config` file
```bash
sudo nano /etc/ssh/sshd_config
```
```bash
# Modify `PermitRootLogin`
PermitRootLogin no

# Modify `PasswordAuthentication`
PasswordAuthentication no

ctrl-x then y then Enter to save and exit
```
Restart SSH config
```bash
sudo service ssh restart
```

#### Continue with setup LAMP Stack? Click [this](https://github.com/Haqimzuhari/Haqimzuhari/blob/master/ubuntu-lamp-stack-terminal.md)