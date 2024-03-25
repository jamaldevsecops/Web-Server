
```
sudo apt-get update
```
```
sudo apt-get upgrade
```
```
sudo apt install nginx
```
```
systemctl start nginx
```
```
systemctl enable nginx
```
```
touch /etc/nginx/sites-available/vhost && nano /etc/nginx/sites-available/vhost
```
```

server {
    listen 80;
    server_name yourdomain.com;
    return 301 https://yourdomain.com;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name yourdomain.com;
    keepalive_timeout 70;

    location / {
        proxy_pass http://your_private_ip:port;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_intercept_errors on;
        error_page 502 = https://error.apsissolutions.com$request_uri;
        }

    ssl_certificate /etc/ssl/certs/yourdomain.com_chain.crt;
    ssl_certificate_key /etc/ssl/certs/yourdomain.com.key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:MozSSL:10m;  # about 40000 sessions
    ssl_session_tickets off;

    # curl https://ssl-config.mozilla.org/ffdhe2048.txt > /path/to/dhparam
    #ssl_dhparam /path/to/dhparam;

    # intermediate configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:DHE-RSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers off;

    # HSTS (ngx_http_headers_module is required) (63072000 seconds)
    add_header Strict-Transport-Security "max-age=63072000" always;
}

```
```
ln -s /etc/nginx/sites-available/vhost /etc/nginx/sites-enabled
```
```
nginx -t
```
```
systemctl restart nginx
```
```
ufw allow 80/tcp
ufw allow 443/tcp 
