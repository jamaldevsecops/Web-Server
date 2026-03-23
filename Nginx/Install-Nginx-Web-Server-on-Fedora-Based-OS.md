# 🚀 Production-Grade Nginx + PHP 8.2 Runbook (Fedora/RHEL-based)

**Author:** Jamal Hossain   
**Date:** 2026-03-22    
**Environment:** RHEL 8 / RHEL 9 / Fedora-based systems    

---

## 📌 Overview
This runbook provides step-by-step instructions to deploy a **production-grade Nginx web server with PHP 8.2**, including:
- Installation
- SELinux & Firewalld configuration
- Virtual Host setup
- SSL (Custom Certificate)
- Performance tuning
- Security hardening
- Troubleshooting

---

## 🧰 Prerequisites
- Root or sudo access
- DNS already configured for:
  - `web1.ngd.com`
- Open ports: 80, 443
- System updated:
```bash
sudo dnf update -y
```

---

# 🟢 1. Nginx Installation
```bash
sudo dnf install -y nginx
sudosystemctl enable --now nginx
sudo systemctl start --now nginx
systemctl status nginx --no-pager
```

---

# 🟢 2. PHP 8.2 Installation (EPEL + REMI)

## 📍 RHEL 8
```bash
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-8.rpm
subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
dnf install -y yum-utils
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.2 -y
sudo dnf install -y php php-fpm php-cli php-mysqlnd php-opcache php-gd php-curl php-mbstring php-xml
```

## 📍 RHEL 9
```bash
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-9.rpm
subscription-manager repos --enable codeready-builder-for-rhel-9-x86_64-rpms
dnf install -y yum-utils
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.2 -y
sudo dnf install -y php php-fpm php-cli php-mysqlnd php-opcache php-gd php-curl php-mbstring php-xml
```

## 📍 Alma Linux 8/9
```bash
EL_VER=$(rpm -E %rhel)
sudo dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-${EL_VER}.noarch.rpm
sudo dnf install -y https://rpms.remirepo.net/enterprise/remi-release-${EL_VER}.rpm
sudo dnf config-manager --set-enabled crb
sudo dnf install -y yum-utils
sudo dnf module reset php -y
sudo dnf module enable php:remi-8.2 -y
sudo dnf install -y php php-fpm php-cli php-mysqlnd php-opcache php-gd php-curl php-mbstring php-xml
```

### ▶️ Enable PHP-FPM
```bash
sudo systemctl enable --now php-fpm
sudo systemctl start --now php-fpm
sudo systemctl status php-fpm --no-pager
```

 ### 🧪 Check the Version
 ```bash
php -v
nginx -v
```

---

# 🔐 3. SELinux Configuration
```bash
sudo setsebool -P httpd_can_network_connect 1
sudo setsebool -P httpd_execmem 1
```

---

# 🔥 4. Firewalld Configuration
```bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
firewall-cmd --list-services
```

---

# 🧪 5. Test with IP:Port
```bash
curl http://SERVER_IP
```
Expected: Nginx default page

---

# 🌐 6. Virtual Host Configuration (web1.ngd.com)

### Create directory
```bash
sudo mkdir -p /var/www/web1.ngd.com/public_html
sudo mkdir -p /var/www/web1.ngd.com/logs
```

### SELinux Configuration && Permissions
```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/web1.ngd.com(/.*)?"
sudo restorecon -Rv /var/www/web1.ngd.com
```

### Create index file
```bash
echo "<?php phpinfo(); ?>" | sudo tee /var/www/web1.ngd.com/public_html/index.php
```

### Nginx Config
```bash
sudo cat > /etc/nginx/conf.d/web1.ngd.com.conf <<'EOF'
server {
    listen 80;
    server_name web1.ngd.com;

    root /var/www/web1.ngd.com/public_html;
    index index.php index.html;

    access_log /var/log/nginx/web1.ngd.com_access.log;
    error_log /var/log/nginx/web1.ngd.com_error.log;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
EOF
```

### Test & Reload
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# 🧪 7. Test HTTP
```bash
curl http://web1.ngd.com
```
Expected: PHP info page

---

# 🔒 8. SSL Configuration (Custom Certificate)

### Copy certificates
```bash
sudo mkdir -p /etc/nginx/ssl
sudo cp your_cert.crt /etc/nginx/ssl/
sudo cp your_private.key /etc/nginx/ssl/
sudo cp ca_bundle.crt /etc/nginx/ssl/
```

### Update Nginx Config
```nginx
server {
    listen 443 ssl http2;
    server_name web1.ngd.com;

    ssl_certificate /etc/nginx/ssl/your_cert.crt;
    ssl_certificate_key /etc/nginx/ssl/your_private.key;
    ssl_trusted_certificate /etc/nginx/ssl/ca_bundle.crt;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    root /var/www/web1.ngd.com/public_html;
    index index.php;

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_pass unix:/run/php-fpm/www.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
}
```

### Reload
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

# 🧪 9. Test HTTPS
```bash
curl -k https://web1.ngd.com
```

---

# 🛡️ 10. Security Hardening

## Pre-check
```bash
sudo nginx -t
sudo ss -tulnp
```

## Disable server tokens
```nginx
server_tokens off;
```

## Security headers
```nginx
add_header X-Frame-Options "SAMEORIGIN";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
```

## File restrictions
```nginx
location ~* \.(env|git) {
    deny all;
}
```

---

# ⚡ 11. Nginx Performance Tuning

Edit:
```bash
sudo nano /etc/nginx/nginx.conf
```

```nginx
worker_processes auto;
worker_connections 4096;
keepalive_timeout 65;

gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

---

# ⚡ 12. PHP Performance Tuning

Edit:
```bash
sudo nano /etc/php-fpm.d/www.conf
```

```ini
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 10
```

Enable OPcache:
```ini
opcache.enable=1
opcache.memory_consumption=128
opcache.max_accelerated_files=10000
```

Restart:
```bash
sudo systemctl restart php-fpm
```

---

# 🧯 13. Troubleshooting

## Check logs
```bash
sudo journalctl -xe
sudo tail -f /var/log/nginx/error.log
```

## Test config
```bash
sudo nginx -t
```

## PHP issues
```bash
sudo systemctl status php-fpm
```

## SELinux issues
```bash
sudo ausearch -m AVC -ts recent
```

---

# ✅ Final Validation Checklist
- [ ] Nginx running
- [ ] PHP-FPM running
- [ ] HTTP working
- [ ] HTTPS working
- [ ] SELinux enforced
- [ ] Firewall open
- [ ] Logs clean

---

# 🎯 Conclusion
You now have a **secure, optimized, production-ready Nginx + PHP 8.2 stack**.

