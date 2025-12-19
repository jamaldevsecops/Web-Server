# 🚀 Laravel + Apache + SELinux — Adjustment

This document describes a **production-grade**, **audit-safe**, and **SELinux-enforcing** setup for running a **Laravel backend** on **RHEL/CentOS 8+** with:

- Apache (`httpd`)
- PHP-FPM
- Remote MySQL (custom port)
- Non-root developer user (`appadmin`)
- Secure shared permissions (no `777`)

---

## 🏗️ Target Architecture

| Component | Value |
|--------|------|
| Web server | Apache (`apache`) |
| PHP runtime | PHP-FPM |
| App owner | `appadmin` (non-root) |
| Shared access | Linux group `webapp` |
| SELinux | Enforcing |
| DB | Remote MySQL (e.g. port `7221`) |
| Writable dirs | `storage`, `bootstrap/cache` |
| Permission model | Group + setgid |
| PHP umask | `002` |

---

## 🔹 STEP 1: Create application user

```bash
useradd -m -s /bin/bash appadmin
passwd appadmin
```

---

## 🔹 STEP 2: Create shared group

```bash
groupadd webapp
usermod -aG webapp apache
usermod -aG webapp appadmin
```

⚠️ **IMPORTANT**: re-login as `appadmin`

```bash
su - appadmin
newgrp webapp
```

---

## 🔹 STEP 3: Deploy application directory

```bash
mkdir -p /var/www/html/nagad-backend-production
chown -R appadmin:webapp /var/www/html/nagad-backend-production
```

Switch to developer user:

```bash
su - appadmin
cd /var/www/html/nagad-backend-production
```

Deploy Laravel source code here.

---

## 🔹 STEP 4: Laravel writable directories

```bash
mkdir -p storage bootstrap/cache
chown -R appadmin:webapp storage bootstrap/cache
chmod -R 2770 storage bootstrap/cache
```

Verify:

```bash
ls -ld storage bootstrap/cache
```

Expected:
```
drwxrws--- appadmin webapp storage
```

---

## 🔹 STEP 5: Ensure parent directory traversal

Apache must traverse **all parent directories**:

```bash
chmod g+x /var /var/www /var/www/html /var/www/html/nagad-backend-production
```

Verify:

```bash
namei -l storage
```

---

## 🔹 STEP 6: SELinux file context (MANDATORY)

Label Laravel writable directories:

```bash
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/nagad-backend-production/storage(/.*)?"
semanage fcontext -a -t httpd_sys_rw_content_t "/var/www/html/nagad-backend-production/bootstrap/cache(/.*)?"

restorecon -Rv /var/www/html/nagad-backend-production
```

Verify:

```bash
ls -Zd storage bootstrap/cache
```

Expected:
```
httpd_sys_rw_content_t
```

---

## 🔹 STEP 7: PHP-FPM umask adjustment (CRITICAL)

Without this step, **group-write breaks after reboot**.

### Edit PHP-FPM pool configuration

```bash
vi /etc/php-fpm.d/www.conf
```

Add the following line:

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

## 🔹 STEP 8: Allow SELinux remote MySQL access

### Enable DB connectivity

```bash
setsebool -P httpd_can_network_connect_db 1
```

### Label custom MySQL port (example: 7221)

```bash
semanage port -a -t mysqld_port_t -p tcp 7221
```

Verify:

```bash
semanage port -l | grep mysqld
```

---

## 🔹 STEP 9: Apache VirtualHost (Backend)

```apache
<VirtualHost *:81>
    ServerName prodweb.nagad.com.bd
    DocumentRoot /var/www/html/nagad-backend-production/public

    <Directory "/var/www/html/nagad-backend-production/public">
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/httpd/nagad-backend-error.log
    CustomLog /var/log/httpd/nagad-backend-access.log combined
</VirtualHost>
```

Reload Apache:

```bash
systemctl reload httpd
```

---

## 🔹 STEP 10: Laravel environment

```bash
su - appadmin
cd /var/www/html/nagad-backend-production

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

## 🔹 STEP 11: Verify Apache write access

```bash
sudo -u apache touch storage/testfile
sudo -u apache rm storage/testfile
```

✔ Must succeed

---

## 🔹 STEP 12: Verify Laravel logging

```bash
sudo -u apache php artisan tinker
```

```php
Log::info('Production setup verified');
```

Check:

```bash
ls storage/logs/laravel.log
```

---

## 🔹 STEP 13: Enforce SELinux

```bash
setenforce 1
getenforce
```

---

## 🔹 STEP 14: Reboot validation

```bash
reboot
```

After reboot:

```bash
getenforce
getsebool httpd_can_network_connect_db
semanage port -l | grep mysqld
```

---

## ✅ Final Production Checklist

- SELinux enforcing
- No world permissions
- Shared group access
- PHP umask correct
- Laravel logs writable
- Survives reboot

---

## 🚫 Never Do in Production

- ❌ `chmod 777`
- ❌ Run Apache as `appadmin`
- ❌ Disable SELinux
- ❌ Assign `httpd_t` to human users

---

## 📌 Key Principle

> Same file access ≠ same SELinux domain

Apache stays confined. Humans stay unprivileged. Files bridge the gap.

---

**End of document**

