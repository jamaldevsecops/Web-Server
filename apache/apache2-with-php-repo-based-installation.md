This guide outlines the installation of Apache and PHP 7.3 on Ubuntu 20.04, followed by enabling `mod_rewrite` for Apache and setting up firewall rules.

### Step-by-Step Breakdown

#### 1. **Install Apache**
   ```bash
   apt-get update
   apt install apache2
   systemctl status apache2
   systemctl enable apache2
   systemctl start apache2
   ```

#### 2. **Install PHP 7.3**
   First, update your system and add the necessary repositories:
   ```bash
   apt-get update
   apt-get install -y language-pack-en-base
   export LC_ALL=en_US.UTF-8 && export LANG=en_US.UTF-8
   apt -y install software-properties-common
   LC_ALL=en_US.UTF-8 add-apt-repository ppa:ondrej/php
   ```

   If needed, add other repositories:
   - For PHP Gearman: `ppa:ondrej/pkg-gearman`
   - For Apache: `ppa:ondrej/apache2`
   - For Nginx: `ppa:ondrej/nginx-mainline` or `ppa:ondrej/nginx`

   Then, install PHP 7.3 and required extensions:
   ```bash
   apt-get update
   apt -y install php7.3
   php -v
   apt-get install -y php7.3-cli php7.3-json php7.3-common php7.3-mysql php7.3-zip php7.3-gd \
                      php7.3-mbstring php7.3-curl php7.3-xml php7.3-bcmath php7.3-bz2 \
                      php7.3-enchant php7.3-gmp php7.3-imap php7.3-intl php7.3-ldap \
                      php7.3-mcrypt php7.3-pspell php7.3-soap php7.3-sqlite3 php7.3-xmlrpc
   php -m
   ```

#### 3. **Modify Apache Configuration for `mod_rewrite`**
   Enable the required modules:
   ```bash
   a2enmod rewrite ssl
   ```

   Edit Apache's configuration to allow `.htaccess` overrides:
   ```bash
   cp /etc/apache2/apache2.conf /etc/apache2/apache2.conf.org
   vim /etc/apache2/apache2.conf
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
   systemctl restart apache2
   ```

#### 4. **Configure Firewall Rules**
   Check the current firewall status and configure the rules:
   ```bash
   ufw status
   ufw allow 22/tcp
   ufw allow 80/tcp
   ufw allow 443/tcp
   ufw enable
   ufw status
   ```

This setup ensures that Apache is installed and configured properly, PHP 7.3 is ready with necessary extensions, and firewall rules are in place for basic web server traffic.