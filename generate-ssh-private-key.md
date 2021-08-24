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