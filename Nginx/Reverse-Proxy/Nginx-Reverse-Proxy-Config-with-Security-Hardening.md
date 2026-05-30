# 🌐 Nginx Reverse Proxy Configuration and Security Hardening Guide for WordPress (Apache Backend)

## 📖 Overview

This guide provides a production-ready Nginx reverse proxy configuration pattern for hosting a WordPress website running on an Apache backend server.

### 🏗️ Architecture

```text
                     Internet
                         │
                         ▼
              ┌───────────────────┐
              │   Nginx Proxy     │
              │ TLS Termination   │
              │ Security Headers  │
              │ Access Control    │
              └─────────┬─────────┘
                        │
                        ▼
              ┌───────────────────┐
              │ Apache Backend    │
              │ PHP-FPM / ModPHP  │
              │ WordPress         │
              └───────────────────┘
```

---

# 🎯 Objectives

- Certificate architecture explained
- TLS 1.2 / TLS 1.3 only
- Strong cipher suites
- OCSP Stapling
- HSTS
- Security headers
- Reverse proxy best practices
- WordPress hardening
- Information disclosure reduction
- Basic attack surface reduction

---

# 🔐 Certificate File Structure

## Server Certificate Components

Typical CA delivery:

```text
domain.crt
intermediate.crt
root.crt
private.key
```

---

# 📜 Generate fullchain.crt

The file served to browsers should contain:

```text
Server Certificate
+
Intermediate Certificate
```

Example:

```bash
cat domain.crt intermediate.crt > fullchain.crt
```

Validate:

```bash
openssl crl2pkcs7 -nocrl -certfile fullchain.crt \
| openssl pkcs7 -print_certs -noout
```

Expected:

```text
Server Certificate
Intermediate Certificate
```

---

# 🛡️ Generate ocsp-chain.crt

Used by:

```nginx
ssl_trusted_certificate
```

Should contain:

```text
Intermediate Certificate
+
Root Certificate
```

Example:

```bash
cat intermediate.crt root.crt > ocsp-chain.crt
```

Validate:

```bash
openssl crl2pkcs7 -nocrl -certfile ocsp-chain.crt \
| openssl pkcs7 -print_certs -noout
```

Expected:

```text
Intermediate Certificate
Root Certificate
```

---

# 📊 SSL Validation

## OpenSSL

```bash
openssl s_client -connect example.com:443 -status
```

## Nmap

```bash
nmap --script ssl-enum-ciphers -p 443 example.com
```

## SSL Labs

Target:

```text
Grade A or A+
Forward Secrecy: Yes
TLS 1.2: Yes
TLS 1.3: Yes
OCSP Stapling: Yes
```

---

# 🔒 TLS Hardening

## Recommended Protocols

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
```

## Recommended Curves

```nginx
ssl_ecdh_curve X25519:prime256v1:secp384r1;
```

## Recommended Cipher Suites

```nginx
ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:
            ECDHE-RSA-AES128-GCM-SHA256:
            ECDHE-ECDSA-AES256-GCM-SHA384:
            ECDHE-RSA-AES256-GCM-SHA384:
            ECDHE-ECDSA-CHACHA20-POLY1305:
            ECDHE-RSA-CHACHA20-POLY1305;

ssl_prefer_server_ciphers on;
```

---

# 🚀 TLS Performance

```nginx
ssl_session_timeout 1d;
ssl_session_cache shared:SSL:50m;
ssl_session_tickets on;
```

Benefits:

- Faster repeat connections
- Reduced TLS handshake overhead
- Better client performance

---

# 🛡️ OCSP Stapling

```nginx
ssl_stapling on;
ssl_stapling_verify on;

resolver 1.1.1.1 1.0.0.1 valid=300s;
resolver_timeout 5s;

ssl_trusted_certificate /path/to/ocsp-chain.crt;
```

Benefits:

- Faster certificate validation
- Reduced CA lookup traffic
- Improved SSL assessment scores

---

# 🔐 HTTP Security Headers

## HSTS

```nginx
add_header Strict-Transport-Security \
"max-age=63072000; includeSubDomains; preload" always;
```

## Clickjacking Protection

```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
```

## MIME Sniffing Protection

```nginx
add_header X-Content-Type-Options "nosniff" always;
```

## Referrer Policy

```nginx
add_header Referrer-Policy \
"strict-origin-when-cross-origin" always;
```

## Permissions Policy

```nginx
add_header Permissions-Policy \
"geolocation=(), microphone=(), camera=()" always;
```

---

# 📜 Content Security Policy

Example CSP:

```nginx
add_header Content-Security-Policy "
default-src 'self';
img-src 'self' data: https:;
script-src 'self' 'unsafe-inline' https:;
style-src 'self' 'unsafe-inline' https:;
font-src 'self' data: https:;
object-src 'none';
frame-ancestors 'self';
base-uri 'self';
" always;
```

⚠️ Test CSP carefully before production rollout.

---

# 🔁 HTTP to HTTPS Redirect

```nginx
server {
    listen 80;
    listen [::]:80;

    return 301 https://$host$request_uri;
}
```

---

# 🌍 Reverse Proxy Configuration

## Backend Proxy

```nginx
location / {

    proxy_pass http://BACKEND_IP:PORT;

    proxy_http_version 1.1;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto https;

    proxy_set_header Connection "";
    proxy_set_header Accept-Encoding "";

    proxy_connect_timeout 30s;
    proxy_send_timeout 30s;
    proxy_read_timeout 30s;
}
```

---

# 🔍 Hide Backend Information

```nginx
server_tokens off;

proxy_hide_header X-Powered-By;
proxy_hide_header Upgrade;
proxy_hide_header Connection;
```

---

# ⏱️ Timeouts

```nginx
client_body_timeout 10s;
client_header_timeout 10s;
keepalive_timeout 15s;
send_timeout 10s;
```

Purpose:

- Reduce Slowloris exposure
- Limit idle connections
- Improve resource utilization

---

# 📦 Upload Limits

```nginx
client_max_body_size 20M;
```

Adjust based on business requirements.

---

# 🔒 WordPress Hardening

## Block wp-config.php

```nginx
location = /wp-config.php {
    deny all;
}
```

## Block XML-RPC

```nginx
location = /xmlrpc.php {
    deny all;
}
```

## Prevent PHP Execution in Uploads

```nginx
location ~* ^/wp-content/uploads/.*\.php$ {
    deny all;
}
```

## Block Readme and License Files

```nginx
location ~* /(readme|license)\.(txt|html)$ {
    deny all;
}
```

---

# 🚫 Sensitive Files Protection

## Hidden Files

```nginx
location ~ /\. {
    deny all;
}
```

## Backup Files

```nginx
location ~* \.(bak|conf|config|dist|ini|log|old|sql|swp|tmp)$ {
    deny all;
}
```

---

# 📊 Logging

```nginx
access_log /var/log/nginx/site_access.log;
error_log  /var/log/nginx/site_error.log warn;
```

Recommended:

- Centralized log collection
- Log rotation
- SIEM integration

---

# ⭐ Production-Ready Reference Configuration

> Replace `example.com`, certificate paths, log paths, and backend IP/port values to match your environment.

```nginx
server {
    listen 80;
    listen [::]:80;

    server_name example.com www.example.com;

    return 301 https://$host$request_uri;
}

server {

    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    server_name example.com www.example.com;

    # Hide nginx version
    server_tokens off;

    access_log /var/log/nginx/site_access.log;
    error_log  /var/log/nginx/site_error.log warn;

    # Certificates
    ssl_certificate     /path/to/fullchain.crt;
    ssl_certificate_key /path/to/private.key;
    ssl_trusted_certificate /path/to/ocsp-chain.crt;

    # TLS Hardening
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ecdh_curve X25519:prime256v1:secp384r1;

    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:
                ECDHE-RSA-AES128-GCM-SHA256:
                ECDHE-ECDSA-AES256-GCM-SHA384:
                ECDHE-RSA-AES256-GCM-SHA384:
                ECDHE-ECDSA-CHACHA20-POLY1305:
                ECDHE-RSA-CHACHA20-POLY1305;

    # Session Resumption
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets on;

    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    resolver 1.1.1.1 1.0.0.1 valid=300s;
    resolver_timeout 5s;

    # Security Headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Content Security Policy
    add_header Content-Security-Policy "default-src 'self'; img-src 'self' data: https:; script-src 'self' 'unsafe-inline' https:; style-src 'self' 'unsafe-inline' https:; font-src 'self' data: https:; object-src 'none'; frame-ancestors 'self'; base-uri 'self';" always;

    # Request Limits
    client_max_body_size 20M;

    # Timeouts
    client_body_timeout 10s;
    client_header_timeout 10s;
    keepalive_timeout 15s;
    send_timeout 10s;

    location / {

        proxy_pass http://BACKEND_IP:PORT;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        proxy_set_header Connection "";
        proxy_set_header Accept-Encoding "";

        proxy_hide_header Upgrade;
        proxy_hide_header Connection;
        proxy_hide_header X-Powered-By;

        proxy_connect_timeout 30s;
        proxy_send_timeout 30s;
        proxy_read_timeout 30s;
    }

    # Block hidden files
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }

    # Block backup/config files
    location ~* \.(bak|conf|config|dist|fla|inc|ini|log|old|psd|sh|sql|swp|tmp)$ {
        deny all;
    }

    # Block wp-config.php
    location = /wp-config.php {
        deny all;
        access_log off;
        log_not_found off;
    }

    # Block XML-RPC
    location = /xmlrpc.php {
        deny all;
        access_log off;
        log_not_found off;
    }

    # Block readme/license files
    location ~* /(readme|license)\.(txt|html)$ {
        deny all;
    }

    # Prevent PHP execution in uploads
    location ~* ^/wp-content/uploads/.*\.php$ {
        deny all;
    }
}
```

---

# 🔎 OCSP Troubleshooting

Verify OCSP URL:

```bash
openssl x509 -in domain.crt -noout -ocsp_uri
```

Check responder:

```bash
curl -I http://ocsp.example-ca.com
```

Validate response:

```bash
openssl ocsp \
-issuer intermediate.crt \
-cert domain.crt \
-url http://ocsp-url-from-certificate \
-text
```

Expected:

```text
Response verify OK
Cert Status: good
```

---


# ✅ Validation Checklist

## TLS

- [ ] TLS 1.0 disabled
- [ ] TLS 1.1 disabled
- [ ] TLS 1.2 enabled
- [ ] TLS 1.3 enabled
- [ ] Strong cipher suites configured
- [ ] OCSP stapling enabled

## Security Headers

- [ ] HSTS enabled
- [ ] CSP configured
- [ ] X-Frame-Options enabled
- [ ] X-Content-Type-Options enabled
- [ ] Referrer Policy configured

## WordPress

- [ ] XML-RPC blocked
- [ ] wp-config.php blocked
- [ ] Upload PHP execution blocked
- [ ] Sensitive files blocked

## Operations

- [ ] Nginx configuration tested
- [ ] SSL Labs scan completed
- [ ] Nmap SSL validation completed
- [ ] Backup procedure documented

---

# 🧪 Useful Validation Commands

```bash
nginx -t
```

```bash
openssl s_client -connect example.com:443 -status
```

```bash
nmap --script ssl-enum-ciphers -p 443 example.com
```

```bash
curl -I https://example.com
```

---

# 🏆 Recommended Security Tooling

- 🔍 Nmap
- 🔒 testssl.sh
- 🛡️ Gixy
- 📋 OpenSCAP
- 🚨 Wazuh
- 📈 SSL Labs

---

**Version:** 1.0

**Audience:** Linux Engineers, DevOps Engineers, Security Engineers, System Administrators

**Scope:** Nginx Reverse Proxy → Apache → WordPress Deployments
