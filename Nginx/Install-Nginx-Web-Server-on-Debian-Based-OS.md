# 🚀 Nginx + PHP-FPM Runbook (Ubuntu 22.04 / 24.04)

## 📌 Overview
This runbook provides step-by-step instructions for deploying:

- 🌐 Nginx Web Server  
- 🐘 PHP-FPM  
- 🔥 UFW Firewall  
- 🔒 SSL (Custom Certificate + CA Bundle)  
- 🛡️ Security Hardening  
- ⚡ Performance Tuning  
- 🧰 Troubleshooting  

---

## 🧱 1. Installation

### 📦 Update system
```bash
sudo apt update && sudo apt upgrade -y
```

### 📦 Install Nginx + PHP-FPM
```bash
sudo apt install -y nginx
```
```bash
sudo apt install -y lsb-release ca-certificates curl
curl -sSLo /tmp/debsuryorg-archive-keyring.deb https://packages.sury.org/debsuryorg-archive-keyring.deb
sudo dpkg -i /tmp/debsuryorg-archive-keyring.deb
echo "deb [signed-by=/usr/share/keyrings/debsuryorg-archive-keyring.gpg] https://packages.sury.org/php/ $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/php.list
sudo apt update && sudo apt install -y php php-fpm php-cli php-common php-mysql php-gd php-mbstring php-xml php-curl php-zip php-opcache
```
---

## ▶️ 2. Enable Services

```bash
sudo systemctl enable --now nginx
sudo systemctl enable --now php8.2-fpm
```

---

## 📁 3. Directory Setup

```bash
sudo mkdir -p /srv/www/web1.ngd.com/public
sudo mkdir -p /var/log/nginx/web1.ngd.com

echo '<?php phpinfo();' | sudo tee /srv/www/web1.ngd.com/public/index.php

sudo chown -R www-data:www-data /srv/www/web1.ngd.com
sudo chmod -R 755 /srv/www/web1.ngd.com
```

---

## 🔥 4. Firewall (UFW)

```bash
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

---

## 🔒 5. SSL Setup

```bash
sudo mkdir -p /etc/nginx/ssl/web1.ngd.com
cat server.crt cabundle.crt > fullchain.crt

sudo chown root:www-data server.key
sudo chmod 640 server.key
sudo chmod 644 fullchain.crt
```

---

## 🌐 6. Nginx Virtual Host

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
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
    }
}
```

---

## ⚡ 7. Nginx Tuning

```nginx
worker_processes auto;
worker_connections 2048;
keepalive_timeout 15;
gzip on;
```

---

## 🐘 8. PHP Tuning

```ini
memory_limit = 256M
upload_max_filesize = 20M
post_max_size = 20M
max_execution_time = 60
expose_php = Off
```

---

## 🧪 9. Validation

```bash
sudo nginx -t
sudo systemctl reload nginx
curl -I https://web1.ngd.com
```

---

## 🧰 10. Troubleshooting

```bash
systemctl status php8.2-fpm
tail -f /var/log/nginx/error.log
```

---

## 🔁 11. Rollback

```bash
sudo rm /etc/nginx/sites-enabled/web1.ngd.com
sudo systemctl reload nginx
```

---

# 🎯 End of Runbook
