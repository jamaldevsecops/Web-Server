Usful Link: https://ssl-config.mozilla.org/#server=nginx&version=1.18.0&config=intermediate&openssl=3.0.2&ocsp=false&guideline=5.7

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
```

### Nginx custom configuraiton: 
```
vim /etc/nginx/nginx.conf
```
```bash
http {

	##
	# Basic Settings
	##
	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
        server_names_hash_bucket_size 128;
        server_names_hash_max_size 512;
	keepalive_timeout 65;
	types_hash_max_size 2048;

        #---Custome Values Added---#
	client_max_body_size 10M;
	server_tokens off;
	fastcgi_hide_header X-Powered-By;
	more_clear_headers Server;
	proxy_connect_timeout       600;
	proxy_send_timeout          600;
	proxy_read_timeout          600;
	send_timeout                600;
	




	
	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1.2 TLSv1.3; # Dropping TLSv1, TLSv1.1, and SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}
```
