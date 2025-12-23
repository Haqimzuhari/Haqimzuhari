# Generate Private SSH key for secure connection
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
|Xoo.= .o. .. |
|=@oo * ..... |
|@*+ o . Eo= |
|X=o o o.* . |
|o+o.o o S o |
|. .. o . . |
| . |
| |
| |
+----[SHA256]-----+
```
This indicate a new pair of SSH key has been generated. You can find the SSH file in `/home/{user}/.ssh` directory
> If permission denied while generating SSH key.<br>Run `sudo chown {user} .ssh`.<br>Try regenerate again<br>
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

---

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
mkdir .ssh
nano ~/.ssh/authorized_keys

ctrl-x then y then Enter to save and exit
```

If `Permission Denied` while creating `.ssh` folder
```bash
sudo mkdir .ssh
sudo chown newuser .ssh
nano ~/.ssh/authorized_keys

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