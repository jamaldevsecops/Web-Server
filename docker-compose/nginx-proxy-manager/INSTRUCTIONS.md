Ref: https://nginxproxymanager.com/setup/#using-mysql-mariadb-database
Create the required content directory. 
```
sudo mkdir nginx-proxy-manager
sudo docker-compose.yml nginx-proxy-manager/
cd nginx-proxy-manager/ sudo docker-compose up -d
```
Browse the Nginx Proxy Manager web GUI: 
```
http: http://192.168.20.126:81
```
Default Administrator Credentials: 
```
Email:    admin@example.com
Password: changeme
```
N.B- If you want to add all http security headers, then follow the below steps:
Ref: https://geekscircuit.com/nginx-proxy-manager

1. add the following docker volume in docker-compose file.
./_hsts.conf:/app/templates/_hsts.conf:ro

2. Create a file called _hsts.conf in your proxy-manager directory and copy paste below code.
```
cd nginx-proxy-manager && vim _hsts.conf
```
```
{% if certificate and certificate_id > 0 -%}
{% if ssl_forced == 1 or ssl_forced == true %}
{% if hsts_enabled == 1 or hsts_enabled == true %}
  # HSTS (ngx_http_headers_module is required) (63072000 seconds = 2 years)
  add_header Strict-Transport-Security "max-age=63072000;{% if hsts_subdomains == 1 or hsts_subdomains == true -%} includeSubDomains;{% endif %} preload" always;
  add_header Referrer-Policy strict-origin-when-cross-origin; 
  add_header X-Content-Type-Options nosniff;
  add_header X-XSS-Protection "1; mode=block";
  add_header X-Frame-Options SAMEORIGIN;
  add_header Content-Security-Policy upgrade-insecure-requests;
  add_header Permissions-Policy interest-cohort=();
  add_header Expect-CT 'enforce; max-age=604800';
  more_set_headers 'Server: Proxy';
  more_clear_headers 'X-Powered-By';
{% endif %}
{% endif %}
{% endif %}
```
```
docker-compose down
docker-compose pull
docker-compose up -d

