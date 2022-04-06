# Linux Ubuntu ODBC Driver 17 for SQL Server OpenSSL Issue
### ODBC Driver 17 for SQL Server SSL Provider unsupported protocol issue
1. Backup original `/etc/ssl/openssl.cnf` to `/etc/ssl/openssl.cnf.backup` if you prefer.
2. Edit `/etc/ssl/openssl.cnf`
```bash
  sudo nano /etc/ssl/openssl.cnf
```
3. On first row:
```bash
  openssl_conf = default_conf
```
4. On last row:
```bash
  [default_conf]
  ssl_conf = ssl_sect

  [ssl_sect]
  system_default = system_default_sect

  [system_default_sect]
  MinProtocol = TLSv1
  CipherString = DEFAULT@SECLEVEL=1
```
5. Restart Apache if using Apache
```bash
  sudo systemctl restart apache2
```
### (Optional: Update to the latest OpenSSL: Not usually working also but can try) Get link of latest version available for OpenSSL from [OpenSSL Official Github release](https://github.com/openssl/openssl/releases)
```bash
# Get latest OpenSSL
# Please change the link using latest version available.
# Current link is just for reference
wget https://www.openssl.org/source/openssl-1.1.1h.tar.gz
tar -zxf openssl-1.1.1h.tar.gz
cd openssl-1.1.1h
./config

# Install dependencies if you not did before
sudo apt-get install make gcc

# Backup
sudo mv /usr/bin/openssl ~/tmp

# Install
sudo make install

# Create symlink to new openssl (if already exists delete it)
sudo ln -s /usr/local/bin/openssl /usr/bin/openssl 

# Update symlinks
sudo ldconfig

# Run verification version
openssl version
# OpenSSL 1.1.1h  22 Sep 2020
```
