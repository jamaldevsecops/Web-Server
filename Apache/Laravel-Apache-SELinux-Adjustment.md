# 🚀 Laravel + Apache + SELinux — Production-Grade Setup (RHEL/CentOS 8+)

This document defines a **secure**, **audit-compliant**, and **SELinux-enforcing** deployment model for running a **Laravel backend** using:

- 🧩 Apache (`httpd`)
- 🐘 PHP-FPM
- 🗄️ Remote MySQL (custom port)
- 👤 Non-root application owner (`appadmin`)
- 🔐 Shared group permissions (no `777`)
- 🛡️ SELinux in **Enforcing** mode

---

## 🏗️ Target Architecture Overview

| Component | Configuration |
|---------|---------------|
| 🌐 Web Server | Apache (`httpd`) |
| 🐘 PHP Runtime | PHP-FPM |
| 👤 App Owner | `appadmin` (non-root) |
| 👥 Shared Group | `webapp` |
| 🛡️ SELinux | Enforcing |
| 🗄️ Database | Remote MySQL (TCP `3306`) |
| ✍️ Writable Paths | `storage`, `bootstrap/cache` |
| 🔑 Permission Model | Group + setgid |
| ⚙️ PHP umask | `002` |

---

## 👤 STEP 1: Create Application User

```bash
useradd -m -s /bin/bash appadmin
passwd appadmin
```

---

## 👥 STEP 2: Create Shared Group

```bash
groupadd webapp
usermod -aG webapp apache
usermod -aG webapp appadmin
```

⚠️ **IMPORTANT**: Re-login to apply group membership

```bash
su - appadmin
newgrp webapp
```

---

## 📁 STEP 3: Deploy Application Directory

```bash
mkdir -p /var/www/html/myapp-backend-production
chown -R appadmin:webapp /var/www/html/myapp-backend-production
```

Switch to developer user and deploy code:

```bash
su - appadmin
cd /var/www/html/myapp-backend-production
```

---

## ✍️ STEP 4: Configure Laravel Writable Directories

```bash
mkdir -p storage bootstrap/cache
chown -R appadmin:webapp storage bootstrap/cache
chmod -R 2770 storage bootstrap/cache
```

Verify permissions:

```bash
ls -ld storage bootstrap/cache
```

Expected:
```
drwxrws--- appadmin webapp storage
```

---

## 🔎 STEP 5: Ensure Parent Directory Traversal

Apache must traverse all parent directories:

```bash
chmod g+x /var /var/www /var/www/html /var/www/html/myapp-backend-production
```

Verify traversal path:

```bash
namei -l storage
```

---

## 🛡️ STEP 6: Apply SELinux File Contexts (MANDATORY)

```bash
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/myapp-backend-production/storage(/.*)?"
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/myapp-backend-production/bootstrap/cache(/.*)?"

restorecon -Rv /var/www/html/myapp-backend-production
```

Verify labels:

```bash
ls -Zd storage bootstrap/cache
```

Expected:
```
httpd_sys_rw_content_t
```

---

## ⚙️ STEP 7: Configure PHP-FPM umask (CRITICAL)

Edit pool configuration:

```bash
vi /etc/php-fpm.d/www.conf
```

Add:
```ini
php_admin_value[umask] = 002
```

Restart services:

```bash
systemctl restart php-fpm httpd
```

Verify umask:

```bash
sudo -u apache php -r 'echo sprintf("%o\n", umask());'
```

Expected:
```
2
```

---

## 🗄️ STEP 8: SELinux Configuration for MySQL (Port 3306)

### 🔍 Verify existing labels
```bash
semanage port -l | grep mysqld
```

### ➕ Add MySQL port
```bash
semanage port -a -t mysqld_port_t -p tcp 3306
```

If already defined:
```bash
semanage port -m -t mysqld_port_t -p tcp 3306
```

### 🔐 Enable DB connectivity
```bash
setsebool -P httpd_can_network_connect_db 1
```

Restart services:
```bash
systemctl restart php-fpm httpd
```

---

## ✔️ STEP 9: Verification Checklist

### 🧪 MySQL Listening
```bash
ss -lntp | grep 3306
```

### 🔐 SELinux Checks
```bash
getenforce
semanage port -l | grep mysqld | grep 3306
getsebool httpd_can_network_connect_db
```

### 🌐 Application Connectivity
```bash
mysql -h 127.0.0.1 -P 3306 -u <db_user> -p
```

Optional PHP test:
```php
<?php
$conn = new mysqli("127.0.0.1", "db_user", "db_password", "db_name", 3306);
if ($conn->connect_error) {
    die("Connection failed: " . $conn->connect_error);
}
echo "MySQL connection successful";
?>
```

---

## 🌐 STEP 10: Apache VirtualHost (Backend)

```apache
<VirtualHost *:81>
    ServerName prodweb.myapp.com.bd
    DocumentRoot /var/www/html/myapp-backend-production/public

    <Directory "/var/www/html/myapp-backend-production/public">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/myapp-backend-error.log
    CustomLog /var/log/httpd/myapp-backend-access.log combined
</VirtualHost>
```

Reload Apache:

```bash
systemctl reload httpd
```

---

## ⚙️ STEP 11: Laravel Environment Setup

```bash
su - appadmin
cd /var/www/html/myapp-backend-production

cp .env.example .env
php artisan key:generate
```

Clear caches:
```bash
php artisan config:clear
php artisan route:clear
php artisan view:clear
```

---

## 🧪 STEP 12: Validate Write Access & Logging

```bash
sudo -u apache touch storage/testfile && rm storage/testfile
```

Laravel log test:
```bash
sudo -u apache php artisan tinker
```
```php
Log::info('Production setup verified');
```

---

## 🔒 STEP 13: Enforce SELinux & Reboot Test

```bash
setenforce 1
reboot
```

Post-reboot validation:
```bash
getenforce
getsebool httpd_can_network_connect_db
semanage port -l | grep mysqld
```

---

## ✅ Final Production Checklist

- 🛡️ SELinux enforcing
- 🔐 No world permissions
- 👥 Shared group access
- ⚙️ PHP umask persistent
- ✍️ Laravel logs writable
- 🔁 Survives reboot

---

## 🚫 Never Do in Production

- ❌ `chmod 777`
- ❌ Run Apache as `appadmin`
- ❌ Disable SELinux
- ❌ Assign `httpd_t` to human users

---

## 📌 Key Principle

> Same file access ≠ Same SELinux domain  
> Apache stays confined. Humans stay unprivileged. Files bridge the gap.
