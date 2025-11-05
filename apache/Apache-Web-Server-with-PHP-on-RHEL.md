# Install Apache, PHP 8.2, and Composer on RHEL 8

This guide covers the installation of **Apache (httpd)**, **PHP 8.2**, and **Composer** (for Laravel setup) on **RHEL 8**. It also includes enabling `mod_rewrite` and configuring firewall rules.

---

## 1. Install Apache

```bash
sudo dnf update -y
sudo dnf install -y httpd
```

Enable and start Apache:
```bash
sudo systemctl enable httpd
sudo systemctl start httpd
sudo systemctl status httpd
```

---

## 2. Install PHP 8.2

RHEL 8 provides PHP versions through **AppStream**.

### Enable PHP 8.2 module
```bash
sudo dnf module reset php -y
sudo dnf module enable php:8.2 -y
```

### Install PHP and common extensions
```bash
sudo dnf install -y php php-cli php-common php-mysqlnd php-json php-gd     php-xml php-mbstring php-curl php-bcmath php-zip php-soap php-intl php-ldap
```

Check PHP version:
```bash
php -v
```

List PHP modules:
```bash
php -m
```

---

## 3. Enable and Configure mod_rewrite

`mod_rewrite` is included with Apache, but you need to allow `.htaccess` overrides.

### Verify the module is loaded:
```bash
sudo httpd -M | grep rewrite
```

If not enabled, uncomment the line in `/etc/httpd/conf/httpd.conf`:
```text
LoadModule rewrite_module modules/mod_rewrite.so
```

### Allow `.htaccess` in web root:

Backup configuration:
```bash
sudo cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.bak
```

Edit Apache config:
```bash
sudo vim /etc/httpd/conf/httpd.conf
```

Find and modify this section:
```text
<Directory "/var/www/html">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

Test and restart Apache:
```bash
sudo apachectl configtest
sudo systemctl restart httpd
```

---

## 4. Configure Firewall Rules

RHEL 8 uses **firewalld** for managing firewall rules.

Open required ports:
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

Check configuration:
```bash
sudo firewall-cmd --list-all
```

---

## 5. Install Composer (for Laravel)

Install dependencies:
```bash
sudo dnf install -y curl php-cli php-zip unzip git
```

Download and install Composer globally:
```bash
curl -sS https://getcomposer.org/installer -o composer-setup.php
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

Verify Composer installation:
```bash
composer -V
```

---

## 6. Test PHP

Create a test PHP file:
```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
```

Open in browser:
```
http://<your-server-ip>/info.php
```

---

## 7. Secure the Server

Remove the PHP info file after testing:
```bash
sudo rm -f /var/www/html/info.php
```

---

## 8. Next Steps: Laravel Setup

Once Composer is installed, you can create a new Laravel project:
```bash
cd /var/www/html
sudo composer create-project laravel/laravel myapp
sudo chown -R apache:apache /var/www/html/myapp
sudo chmod -R 775 /var/www/html/myapp/storage
```

Visit the application in a browser:
```
http://<your-server-ip>/myapp/public
```

---

### ✅ Summary

| Component | Command/Action |
|------------|----------------|
| Apache install | `sudo dnf install httpd -y` |
| PHP install | `sudo dnf module enable php:8.2 -y && sudo dnf install php -y` |
| mod_rewrite | Set `AllowOverride All` in Apache config |
| Firewall | `sudo firewall-cmd --add-service=http/https --permanent --reload` |
| Composer | `curl -sS https://getcomposer.org/installer ...` |
| Laravel setup | `composer create-project laravel/laravel myapp` |
