# 🔐 Production-Grade Nginx + PHP 8.2 Runbook (CIS-Aligned)

---

## 📌 Overview
This runbook aligns with **CIS (Center for Internet Security) Benchmarks** and industry best practices for:
- Nginx Hardening
- PHP Security
- OS-level controls (SELinux, Firewalld)

---

# 🛡️ CIS Benchmark Alignment Summary

| Area | Control | Status |
|------|--------|--------|
| OS Hardening | SELinux Enforcing | ✅ |
| Network Security | Firewalld Restrictive Rules | ✅ |
| Web Server | Hide version info | ✅ |
| Web Server | Secure headers | ✅ |
| TLS Security | Strong protocols & ciphers | ✅ |
| File Permissions | Least privilege | ✅ |
| Logging | Enabled & monitored | ✅ |

---

# 🔐 1. OS Hardening (CIS Level 1)

## Disable unused services
```bash
sudo systemctl disable postfix
sudo systemctl disable avahi-daemon
```

## Ensure SELinux enforcing
```bash
getenforce
sudo setenforce 1
```

---

# 🔐 2. Nginx CIS Hardening

## Hide version
```nginx
server_tokens off;
```

## Disable unused HTTP methods
```nginx
if ($request_method !~ ^(GET|HEAD|POST)$ ) {
    return 444;
}
```

## Security Headers
```nginx
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Content-Security-Policy "default-src 'self';" always;
```

---

# 🔐 3. TLS Hardening (CIS Critical)

```nginx
ssl_protocols TLSv1.2 TLSv1.3;
ssl_prefer_server_ciphers on;
ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
ssl_session_tickets off;
```

---

# 🔐 4. File System Security

```bash
sudo chown -R nginx:nginx /var/www
sudo chmod -R 750 /var/www
```

Restrict sensitive files:
```nginx
location ~* \.(env|ini|log|conf)$ {
    deny all;
}
```

---

# 🔐 5. PHP Security Hardening

Edit `/etc/php.ini`:

```ini
expose_php = Off
display_errors = Off
log_errors = On
allow_url_fopen = Off
allow_url_include = Off
```

Disable dangerous functions:
```ini
disable_functions = exec,passthru,shell_exec,system
```

---

# 🔐 6. Logging & Monitoring (CIS Requirement)

## Enable access & error logs
```nginx
access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log warn;
```

## Log rotation
```bash
sudo dnf install logrotate -y
```

---

# 🔐 7. Firewall Hardening

Allow only required ports:
```bash
sudo firewall-cmd --permanent --remove-service=ssh
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

---

# 🔐 8. Rate Limiting & DDoS Protection

```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

location / {
    limit_req zone=one burst=20;
}
```

---

# 🔐 9. Fail2Ban (Recommended)

```bash
sudo dnf install fail2ban -y
sudo systemctl enable fail2ban
sudo systemctl start fail2ban
```

---

# 🔐 10. Pre & Post CIS Validation

## Pre-check
```bash
sudo nginx -t
sudo ss -tulnp
```

## Post-check tools
```bash
sudo dnf install lynis -y
sudo lynis audit system
```

---

# 🎯 Final CIS Checklist

- [ ] SELinux Enforcing
- [ ] Firewall restricted
- [ ] TLS 1.2+ only
- [ ] No server tokens
- [ ] Secure headers enabled
- [ ] PHP hardened
- [ ] Logs enabled
- [ ] Rate limiting configured

---

# 🚀 Conclusion
This configuration ensures your stack meets **CIS-aligned security standards**, reducing attack surface and improving compliance readiness.
