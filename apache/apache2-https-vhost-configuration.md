# Apache2 HTTPS Virtual Host Configuration 

```bash
sudo a2enmod headers
```

```bash
# generated 2025-01-02, Mozilla Guideline v5.7, Apache 2.4.62, OpenSSL 1.1.1f (UNSUPPORTED; end-of-life), modern config
# https://ssl-config.mozilla.org/#server=apache&version=2.4.62&config=modern&openssl=1.1.1f&guideline=5.7

# this configuration requires mod_ssl, mod_rewrite, mod_headers, and mod_socache_shmcb
<VirtualHost *:80>
    RewriteEngine On
    RewriteCond %{REQUEST_URI} !^/.well-known/acme-challenge/
    RewriteRule ^.*$ https://%{HTTP_HOST}%{REQUEST_URI} [R=301,QSA,L]
</VirtualHost>

<VirtualHost *:443>
    SSLEngine on
    ServerAdmin infra@apsissolutions.com
    DocumentRoot /var/www/html
    ServerName apsissolutions.com
    ServerAlias www.apsissolutions.com
    SSLCertificateFile /etc/ssl/certs/apsisweb/2024/apsissolutions.com.crt
    SSLCertificateKeyFile /etc/ssl/certs/apsisweb/2024/apsissolutions.com.key
    SSLCertificateChainFile /etc/ssl/certs/apsisweb/2024/apsissolutions_Bundle.crt

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined	
    
    # enable HTTP/2, if available
    Protocols h2 http/1.1

    # HTTP Security Headers
    Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"
    Header set Content-Security-Policy "default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self'; font-src 'self';"
    Header always set X-Frame-Options "SAMEORIGIN"
    Header always set X-XSS-Protection "0"
    Header always set X-Content-Type-Options "nosniff"

    # OCSP Stapling Configuration
    SSLUseStapling on
    SSLStaplingResponderTimeout 5
    SSLStaplingReturnResponderErrors off
    
</VirtualHost>

#Information Leakage
ServerTokens Prod
ServerSignature Off

#SSL/TLS Configuration
# modern configuration
SSLProtocol -all +TLSv1.2 +TLSv1.3
SSLHonorCipherOrder On
SSLCipherSuite TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:TLS_AES_128_GCM_SHA256:\
ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:\
ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-CHACHA20-POLY1305:\
ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES128-GCM-SHA256

SSLOpenSSLConfCmd       Curves X25519:prime256v1:secp384r1
SSLHonorCipherOrder     off
SSLSessionTickets       off
SSLUseStapling On
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
SSLInsecureRenegotiation off
SSLCompression off

#Denial of Service Mitigations
Timeout 10
KeepAlive On
MaxKeepAliveRequests 100
KeepAliveTimeout 15
RequestReadTimeout header=20-40,MinRate=500 body=20,MinRate=500

#Request Limits
LimitRequestline 8190
LimitRequestFields 100
LimitRequestFieldsize 1024
LimitRequestBody 102400
```

```bash
apachectl configtest
```

```bash
sudo systemctl restart apache2
```

