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

    # HTTP Strict Transport Security (mod_headers is required) (63072000 seconds)
    Header always set Strict-Transport-Security "max-age=63072000"
</VirtualHost>

# modern configuration
SSLProtocol             -all +TLSv1.3
SSLOpenSSLConfCmd       Curves X25519:prime256v1:secp384r1
SSLHonorCipherOrder     off
SSLSessionTickets       off

SSLUseStapling On
SSLStaplingCache "shmcb:logs/ssl_stapling(32768)"
```

```bash
apachectl configtest
```

```bash
sudo systemctl restart apache2
```

