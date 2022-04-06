# Linux Ubuntu ODBC Driver 17 for SQL Server OpenSSL Issue
### ODBC Driver 17 for SQL Server SSL Provider unsupported protocol issue
1. Backup original /etc/ssl/openssl.cnf
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
### Get link of latest version available for OpenSSL from [OpenSSL Official Github release](https://github.com/openssl/openssl/releases)
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
