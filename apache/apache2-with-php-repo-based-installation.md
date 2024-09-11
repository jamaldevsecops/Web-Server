# Install Apache with PHP on Ubuntu 20.04
This guide outlines the installation of Apache and PHP 7.3 on Ubuntu 20.04, followed by enabling `mod_rewrite` for Apache and setting up firewall rules.

### Step-by-Step Breakdown

#### 1. **Install Apache**
   ```bash
   sudo apt-get update
   sudo apt -y install apache2
   ```
   ```bash
   sudo systemctl enable apache2
   sudo systemctl start apache2
   sudo systemctl status apache2
   ```

#### 2. **Install PHP 7.3**
   First, update your system and add the necessary repositories:
   ```bash
   sudo apt-get update
   sudo apt -y install language-pack-en-base &&  sudo apt -y install software-properties-common
   ```
   ```bash
   sudo export LC_ALL=en_US.UTF-8 && export LANG=en_US.UTF-8 
   sudo LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
   ```

   If needed, add other repositories:
   - For PHP Gearman: `ppa:ondrej/pkg-gearman`
   - For Apache: `ppa:ondrej/apache2`
   - For Nginx: `ppa:ondrej/nginx-mainline` or `ppa:ondrej/nginx`

   Then, install PHP 7.3 and required extensions:
   ```bash
   sudo apt-get update
   sudo apt -y install php7.3
   php -v
   sudo apt-get install -y php7.3-cli php7.3-json php7.3-common php7.3-mysql php7.3-zip php7.3-gd \
                      php7.3-mbstring php7.3-curl php7.3-xml php7.3-bcmath php7.3-bz2 \
                      php7.3-enchant php7.3-gmp php7.3-imap php7.3-intl php7.3-ldap \
                      php7.3-mcrypt php7.3-pspell php7.3-soap php7.3-sqlite3 php7.3-xmlrpc
   php -m
   ```

#### 3. **Modify Apache Configuration for `mod_rewrite`**
   Enable the required modules:
   ```bash
   sudo a2enmod rewrite ssl
   ```

   Edit Apache's configuration to allow `.htaccess` overrides:
   ```bash
   sudo cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.org
   sudo vim /etc/apache2/apache2.conf
   ```

   Update the `<Directory /var/www/>` block:
   ```text
   <Directory /var/www/>
       Options Indexes FollowSymLinks
       AllowOverride all    # Change 'none' to 'all'
       Require all granted
   </Directory>
   ```

   Save and restart Apache:
   ```bash
   sudo apachectl configtest
   sudo systemctl restart apache2
   ```

#### 4. **Configure Firewall Rules**
   Check the current firewall status and configure the rules:
   ```bash
   sudo ufw status
   sudo ufw allow 80/tcp && sudo ufw allow 443/tcp
   sudo ufw enable
   sudo ufw status
   ```

This setup ensures that Apache is installed and configured properly, PHP 7.3 is ready with necessary extensions, and firewall rules are in place for basic web server traffic.
