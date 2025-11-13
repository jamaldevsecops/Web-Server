# 🔐 Nginx Security Hardening Guide

## Based on CIS Benchmark Recommendations

This document provides a **complete Nginx hardening guide** following
CIS (Center for Internet Security) best practices.

------------------------------------------------------------------------

## 🛡 1. File & Directory Permissions (CIS 3.x)

### ✔ Ensure correct ownership

``` bash
sudo chown -R root:root /etc/nginx
sudo chown -R root:root /var/log/nginx
```

### ✔ Restrict permissions

``` bash
sudo chmod -R 750 /etc/nginx
sudo chmod -R 750 /var/log/nginx
```

------------------------------------------------------------------------

## 🛡 2. Run Nginx as a Non-Privileged User (CIS 2.x)

Edit `/etc/nginx/nginx.conf`:

``` nginx
user nginx;  # or www-data (Ubuntu)
```

Ensure user exists:

``` bash
id nginx
```

------------------------------------------------------------------------

## 🛡 3. Disable Unnecessary Modules (CIS 4.x)

When building Nginx from source, use:

    --without-http_autoindex_module
    --without-http_ssi_module
    --without-http_userid_module
    --without-http_scgi_module

------------------------------------------------------------------------

## 🛡 4. Enable Only Secure TLS Versions (CIS 5.x)

Inside `server {}`:

``` nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;

ssl_ciphers HIGH:!aNULL:!MD5:!3DES;
```

------------------------------------------------------------------------

## 🛡 5. Configure Strong Diffie-Hellman Keys (CIS 5.x)

``` bash
sudo openssl dhparam -out /etc/nginx/dhparam.pem 4096
```

Add:

``` nginx
ssl_dhparam /etc/nginx/dhparam.pem;
```

------------------------------------------------------------------------

## 🛡 6. Disable Weak Headers (CIS 6.x)

Inside `http {}`:

``` nginx
server_tokens off;
more_clear_headers Server;
more_clear_headers X-Powered-By;
```

If `headers-more` is not installed:

``` bash
sudo apt install nginx-extras
```

------------------------------------------------------------------------

## 🛡 7. Add Security Headers

Inside `http {}` or `server {}`:

``` nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
add_header Permissions-Policy "geolocation=(), microphone=()" always;
add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
```

------------------------------------------------------------------------

## 🛡 8. Enable Access Control for Admin Areas

``` nginx
location /admin {
    allow 192.168.1.0/24;
    deny all;
}
```

------------------------------------------------------------------------

## 🛡 9. Disable Unused HTTP Methods

``` nginx
if ($request_method !~ ^(GET|POST|HEAD)$ ) {
    return 405;
}
```

------------------------------------------------------------------------

## 🛡 10. Protect Against Slowloris (CIS DoS Hardening)

``` nginx
client_body_timeout 10;
client_header_timeout 10;
send_timeout 10;
keepalive_timeout 10;
```

------------------------------------------------------------------------

## 🛡 11. Enable Rate Limiting

Inside `http {}`:

``` nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;
```

Inside `server {}`:

``` nginx
limit_req zone=one burst=20 nodelay;
```

------------------------------------------------------------------------

## 🛡 12. Restrict Buffer Sizes to Avoid Buffer Overflows

``` nginx
client_body_buffer_size 1k;
client_max_body_size 1m;
large_client_header_buffers 2 1k;
```

------------------------------------------------------------------------

## 🛡 13. Hide Directory Listing

``` nginx
autoindex off;
```

------------------------------------------------------------------------

## 🛡 14. Use Strict Logging Permissions

``` bash
sudo chmod 640 /var/log/nginx/access.log
sudo chmod 640 /var/log/nginx/error.log
```

------------------------------------------------------------------------

## 🛡 15. Enable Real-Time Log Monitoring

Tools: - fail2ban\
- Wazuh\
- OSSEC

Example with Fail2Ban:

    sudo apt install fail2ban

------------------------------------------------------------------------

## 🛡 16. Validate Nginx Configuration Regularly

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

------------------------------------------------------------------------

## 🛡 17. Keep Nginx Up to Date

Ubuntu:

``` bash
sudo apt update && sudo apt upgrade nginx -y
```

RHEL:

``` bash
sudo dnf upgrade nginx -y
```

------------------------------------------------------------------------

## 🏁 NGINX CIS HARDENING COMPLETE

This guide includes: ✔ TLS Hardening\
✔ Security Headers\
✔ Rate Limiting\
✔ DoS Protections\
✔ Permission Hardening\
✔ Module Minimization\
✔ TLS Recommendations\
✔ Logging Security\
✔ Access Control
