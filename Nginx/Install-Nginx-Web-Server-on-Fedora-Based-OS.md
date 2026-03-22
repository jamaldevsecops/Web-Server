# 🚀 Nginx + PHP-FPM Runbook (RHEL 9 / Rocky / AlmaLinux)

## 📌 Overview
This runbook provides step-by-step instructions for deploying **Nginx + PHP-FPM** with:
- 🔐 SELinux (Enforcing)
- 🔥 firewalld
- 🌐 Virtual Host (web1.ngd.com)
- 🔒 SSL (Custom Certificate + CA Bundle)
- 🛡️ Security Hardening
- ⚡ Performance Tuning
- 🧰 Troubleshooting

---

## 🧱 1. Installation
```bash
sudo dnf -y update && dnf -y install nginx
```
### 📦 Install packages
```bash
sudo dnf install -y epel-release
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.2 -y
sudo dnf install -y php php-fpm php-cli php-common php-mysqlnd php-gd php-mbstring php-xml php-opcache
```

### ▶️ Enable services
```bash
systemctl enable --now nginx php-fpm
```

---

## 📁 2. Directory Setup

```bash
mkdir -p /srv/www/web1.ngd.com/public
mkdir -p /var/log/nginx/web1.ngd.com
echo '<?php phpinfo();' > /srv/www/web1.ngd.com/public/index.php
chown -R nginx:nginx /srv/www/web1.ngd.com
chmod -R 0755 /srv/www/web1.ngd.com
```

---

## 🔐 3. SELinux Configuration

### 📌 Apply context
```bash
semanage fcontext -a -t httpd_sys_content_t "/srv/www/web1.ngd.com(/.*)?"
restorecon -Rv /srv/www/web1.ngd.com
```

### ✏️ Writable directory
```bash
mkdir -p /srv/www/web1.ngd.com/storage
chown -R nginx:nginx /srv/www/web1.ngd.com/storage
semanage fcontext -a -t httpd_sys_rw_content_t "/srv/www/web1.ngd.com/storage(/.*)?"
restorecon -Rv /srv/www/web1.ngd.com/storage
```

### 🌐 Allow outbound connections (if needed)
```bash
setsebool -P httpd_can_network_connect on
```

---

## 🔥 4. Firewall Configuration

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

---

## 🔒 5. SSL Setup

### 📂 Directory
```bash
mkdir -p /etc/nginx/ssl/web1.ngd.com
```

### 🔗 Combine certificate chain
```bash
cat server.crt cabundle.crt > fullchain.crt
```

### 🔐 Secure permissions
```bash
chown root:nginx server.key
chmod 640 server.key
chmod 644 fullchain.crt
```

---

## 🌐 6. Nginx Virtual Host

📄 `/etc/nginx/conf.d/web1.ngd.com.conf`

```nginx
server {
    listen 80;
    server_name web1.ngd.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    server_name web1.ngd.com;

    root /srv/www/web1.ngd.com/public;
    index index.php;

    ssl_certificate /etc/nginx/ssl/web1.ngd.com/fullchain.crt;
    ssl_certificate_key /etc/nginx/ssl/web1.ngd.com/server.key;

    add_header Strict-Transport-Security "max-age=31536000" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-Frame-Options "SAMEORIGIN" always;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

---

## ⚡ 7. Nginx Performance Tuning

📄 `/etc/nginx/nginx.conf`

```nginx
worker_processes auto;
worker_connections 2048;

keepalive_timeout 15;
keepalive_requests 100;

sendfile on;
tcp_nopush on;
tcp_nodelay on;

gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

---

## 🐘 8. PHP-FPM Tuning

📄 `/etc/php-fpm.d/www.conf`

```ini
pm = ondemand
pm.max_children = 20
pm.process_idle_timeout = 10s
pm.max_requests = 500
```

📄 `/etc/php.ini`

```ini
memory_limit = 256M
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 60

opcache.enable=1
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

---

## 🛡️ 9. Security Hardening

- ❌ Disable version exposure
```nginx
server_tokens off;
```

- 🔒 Secure PHP
```ini
expose_php = Off
display_errors = Off
```

- 🚫 Block hidden files
```nginx
location ~ /\. {
    deny all;
}
```

---

## 🧪 10. Validation

```bash
nginx -t
systemctl reload nginx
curl -I https://web1.ngd.com
```

---

## 🧰 11. Troubleshooting

### ❗ 502 Bad Gateway
```bash
systemctl status php-fpm
```

### ❗ SELinux issues
```bash
ausearch -m avc -ts recent
```

### ❗ Firewall issues
```bash
firewall-cmd --list-all
```

---

## 🔁 12. Rollback

```bash
mv /etc/nginx/conf.d/web1.ngd.com.conf /tmp/
nginx -t
systemctl reload nginx
```

---

## 📎 Notes
- ✅ Keep SELinux enforcing
- 🔐 Always protect private keys
- 🔄 Regular updates recommended

---

# 🎯 End of Runbook
