# Linux Fix General Issue (Linux Mint)
## Fix Fn key of External Keyboard
###### The issue is, media shortcut is binded on FN keys. Its override everything such as when trying to use VSCode, you sometime just click F1 to toggle command or F2 to rename file. This media shortcut has override the function and pissed you off. So we remove them permanently.
###### The issue heppened because Linux thought the external keyboard is an Apple keyboard.
1.  Create and edit below file
```bash
sudo nano /etc/modprobe.d/hid_apple.conf
```
2.  Add below command into the file the save and exit
```bash
options hid_apple fnmode=2
```
3. Execute below command
```bash
sudo update-initramfs -u
```
4. Reboot. Done

## Switch default PHP version
###### This issue is when your existing Laravel project is still using Laravel 8(PHP7) but your Composer is listening to PHP8
###### It is still no solution to convert Laravel project from one version to another easily.
###### The best way is install multiple PHP version (PHP7 & PHP8), then just change with PHP is going to be default for Composer to listen to.
Just open terminal and run below command
```bash
sudo update-alternatives --config php
```
Then just pick which PHP that going to be listen by Composer
When its done, check current default PHP version
```bash
php -v
```
And check also which PHP our Composer listening to
```bash
composer -vvv about
```
##### That pretty much it.
