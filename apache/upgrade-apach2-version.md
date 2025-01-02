# Upgrade Apache2 Version
The ondrej/apache2 PPA (Personal Package Archive) provides newer versions of Apache for Ubuntu.  
Step 1 - Add the PPA:
```bash
sudo add-apt-repository ppa:ondrej/apache2
sudo apt update
```
Step 2 - Upgrade Apache: After adding the PPA, upgrade your Apache installation:
```bash
sudo apt upgrade apache2
```
Step 3 - Verify the Version: Confirm that Apache is now upgraded:
```bash
apache2 -v
```
## Install Apache 2.4.61 from Source (If Necessary)
If the PPA does not provide Apache 2.4.61 or you prefer to build it manually:

Steps:
Install Dependencies: Install the required tools and libraries:
```bash
sudo apt update
sudo apt install -y build-essential libpcre3 libpcre3-dev libssl-dev zlib1g-dev
```

Download Apache Source Code: Visit the Apache HTTP Server download page and download the 2.4.61 tarball. Alternatively, use wget:
```bash
wget https://downloads.apache.org//httpd/httpd-2.4.61.tar.gz
```

Extract the Archive:
```bash
tar -xvzf httpd-2.4.61.tar.gz
cd httpd-2.4.61
```

Configure and Build Apache: Run the following commands to configure and compile:
```bash
./configure --enable-so --enable-ssl --with-mpm=prefork
make
sudo make install
```

Restart Apache:
```bash
sudo systemctl restart apache2
```
Verify the Installation:
```bash
/usr/local/apache2/bin/httpd -v
```

Before upgrading, ensure you back up your current Apache configuration and test the updated installation in a staging environment:
```bash
sudo cp -r /etc/apache2 /etc/apache2.bak
```
