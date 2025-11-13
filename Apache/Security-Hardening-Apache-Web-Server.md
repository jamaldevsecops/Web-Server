# 🔐 Apache Security Hardening Guide

## Based on CIS Benchmark

------------------------------------------------------------------------

# 🛡 1. Correct Ownership & Permissions

``` bash
sudo chown -R root:root /etc/httpd
sudo chown -R root:root /etc/apache2
sudo chmod -R 750 /etc/httpd
sudo chmod -R 750 /etc/apache2
```

------------------------------------------------------------------------

# 🛡 2. Disable Server Signature & Tokens

Edit:

Ubuntu → `/etc/apache2/conf-available/security.conf`\
RHEL → `/etc/httpd/conf/httpd.conf`

``` apache
ServerSignature Off
ServerTokens Prod
```

------------------------------------------------------------------------

# 🛡 3. Disable Unused Apache Modules

Ubuntu:

``` bash
sudo a2dismod autoindex status negotiation cgi
```

RHEL: edit `/etc/httpd/conf.modules.d/*.conf` and comment modules.

------------------------------------------------------------------------

# 🛡 4. Enable Strong SSL/TLS

``` apache
SSLProtocol TLSv1.2 TLSv1.3
SSLCipherSuite HIGH:!aNULL:!MD5:!3DES
SSLHonorCipherOrder on
```

Generate strong DH key:

``` bash
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 4096
```

------------------------------------------------------------------------

# 🛡 5. Security Headers

``` apache
Header always set X-Frame-Options "SAMEORIGIN"
Header always set X-XSS-Protection "1; mode=block"
Header always set X-Content-Type-Options "nosniff"
Header always set Referrer-Policy "strict-origin-when-cross-origin"
Header always set Permissions-Policy "geolocation=(), microphone=()"
Header always set Strict-Transport-Security "max-age=63072000"
```

------------------------------------------------------------------------

# 🛡 6. Limit HTTP Methods

``` apache
<LimitExcept GET POST HEAD>
    Deny from all
</LimitExcept>
```

------------------------------------------------------------------------

# 🛡 7. Enable ModSecurity (WAF)

Ubuntu:

``` bash
sudo apt install libapache2-mod-security2
sudo a2enmod security2
```

RHEL:

``` bash
sudo dnf install mod_security
```

Enable OWASP CRS:

``` bash
sudo cp /usr/share/modsecurity-crs/base_rules/* /etc/modsecurity/
```

------------------------------------------------------------------------

# 🛡 8. Rate Limiting with mod_ratelimit

``` apache
<IfModule mod_ratelimit.c>
    SetOutputFilter RATE_LIMIT
    SetEnv rate-limit 400
</IfModule>
```

------------------------------------------------------------------------

# 🛡 9. Disable Directory Listing

``` apache
Options -Indexes
```

------------------------------------------------------------------------

# 🛡 10. Logging Permissions

``` bash
sudo chmod 640 /var/log/apache2/*.log
sudo chmod 640 /var/log/httpd/*.log
```

------------------------------------------------------------------------

# 🛡 11. Fail2Ban Protection

Install:

``` bash
sudo apt install fail2ban
```

Apache rules:

``` bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

------------------------------------------------------------------------

# 🛡 12. Keep Apache Updated

Ubuntu:

``` bash
sudo apt update && sudo apt upgrade apache2 -y
```

RHEL:

``` bash
sudo dnf upgrade httpd -y
```
