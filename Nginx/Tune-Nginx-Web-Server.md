# ⚡ Nginx High-Performance Tuning (Low Latency & High Traffic)

This document includes a **custom configuration file** instead of
modifying the default `nginx.conf`.

------------------------------------------------------------------------

## 🎯 1. Why Use a Custom Tuning File?

Using a custom config file:

✔ Keeps `/etc/nginx/nginx.conf` clean\
✔ Prevents conflicts during upgrades\
✔ Makes tuning modular and maintainable\
✔ Allows you to apply changes without touching core files

------------------------------------------------------------------------

## 📁 2. Create a Custom Nginx Configuration Directory

``` bash
sudo mkdir -p /etc/nginx/custom.d
```

------------------------------------------------------------------------

## 📝 3. Modify nginx.conf to Use Custom Includes

Edit `/etc/nginx/nginx.conf` and add inside the `http {}` block:

``` nginx
include /etc/nginx/custom.d/*.conf;
```

Example recommended base `nginx.conf`:

``` nginx
user nginx;
worker_processes auto;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    include /etc/nginx/conf.d/*.conf;

    # Custom performance tuning directory
    include /etc/nginx/custom.d/*.conf;
}
```

------------------------------------------------------------------------

## ⚙️ 4. Create Your Custom Tuning File

Create:

    /etc/nginx/custom.d/tuning.conf

Add all tuning parameters:

``` nginx
# -------------------------------------
#  NGINX PERFORMANCE TUNING
# -------------------------------------

worker_rlimit_nofile 200000;

# EVENTS tuning
events {
    worker_connections 65535;
    multi_accept on;
    use epoll;
}

# HTTP performance tuning
sendfile on;
tcp_nopush on;
tcp_nodelay on;

keepalive_timeout 15;
keepalive_requests 10000;

client_body_buffer_size 1m;
client_header_buffer_size 1k;
large_client_header_buffers 4 8k;

client_max_body_size 50m;

send_timeout 10;
client_body_timeout 10;
client_header_timeout 10;

# GZIP optimization
gzip on;
gzip_disable "msie6";
gzip_comp_level 5;
gzip_min_length 256;
gzip_proxied any;
gzip_types
  text/plain
  text/css
  application/json
  application/javascript
  text/xml
  application/xml
  application/xml+rss
  image/svg+xml;

# Static file caching
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    access_log off;
}

# Proxy caching
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mycache:100m inactive=60m;
proxy_cache mycache;
proxy_cache_use_stale error timeout invalid_header updating;
```

------------------------------------------------------------------------

## ⚠️ 5. IMPORTANT: Prevent Duplicate `events {}` Blocks

Since your custom config includes:

``` nginx
events {
    worker_connections 65535;
    multi_accept on;
    use epoll;
}
```

You must comment out the **default** one inside `nginx.conf`:

``` nginx
# events {
#     worker_connections 1024;
# }
```

Only **one** `events {}` block is allowed.

------------------------------------------------------------------------

## 🧠 6. Kernel Network Tuning (sysctl)

Edit `/etc/sysctl.conf`:

``` bash
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_max_syn_backlog = 4096
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
```

Apply with:

``` bash
sudo sysctl -p
```

------------------------------------------------------------------------

## 🌐 7. Enable HTTP/2

``` nginx
server {
    listen 443 ssl http2;
    ...
}
```

------------------------------------------------------------------------

## 🤝 8. Load Balancer Example

``` nginx
upstream backend {
    zone backend 64k;
    least_conn;
    server app01 weight=1;
    server app02 weight=1;
}
```

------------------------------------------------------------------------

## 🧹 9. Logging Optimization

``` nginx
access_log off;
```

or:

``` nginx
access_log /var/log/nginx/access.log main buffer=512k flush=1m;
```

------------------------------------------------------------------------

## 🧪 10. Benchmark Tools

### 🚀 wrk

``` bash
wrk -t8 -c2000 -d30s http://yourserver/
```

### 📊 Apache Benchmark

``` bash
ab -n 100000 -c 2000 http://yourserver/
```

### 🛡 Siege

``` bash
siege -c 2000 -t 1M http://yourserver/
```

------------------------------------------------------------------------

## ▶️ 11. Test & Reload Nginx

``` bash
sudo nginx -t
sudo systemctl reload nginx
```

------------------------------------------------------------------------

## 🎉 Your custom configuration tuning is fully ready!

This file contains: ✔ Custom include method\
✔ ALL tuning settings\
✔ Caching configs\
✔ Gzip configs\
✔ Kernel tuning\
✔ Worker tuning\
✔ Logging optimization\
✔ Load balancer example\
✔ Benchmark tools
