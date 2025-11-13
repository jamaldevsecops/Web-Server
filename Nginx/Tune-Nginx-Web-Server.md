# ⚡ Nginx High-Performance Tuning (Low Latency & High Traffic)

### 🧪 Benchmark Report Before Any Tunning (default)
```
wrk -t2 -c10 -d10s http://192.168.20.126/
Running 10s test @ http://192.168.20.126/
  2 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     1.18ms    2.27ms  46.26ms   98.62%
    Req/Sec     4.83k     1.05k    7.65k    75.00%
  96218 requests in 10.01s, 78.82MB read
Requests/sec:   9615.67
Transfer/sec:      7.88MB
```

## 📝 1. Modify nginx.conf to Use Custom Configuration

Take `/etc/nginx/nginx.conf` file as backup and add the following example:

``` nginx
cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.original
vim /etc/nginx/nginx.conf
```
Example recommended base `nginx.conf`:

``` nginx
user www-data;
worker_processes auto;
worker_cpu_affinity auto;
worker_rlimit_nofile 200000;

pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 65535;
    multi_accept on;
    use epoll;
}

http {

    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    server_tokens off;
    more_clear_headers Server;
    more_clear_headers X-Powered-By;
    more_set_headers "X-Content-Type-Options: nosniff";

    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # SSL Hardening
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;

    # Logging
    access_log /var/log/nginx/access.log;
    error_log  /var/log/nginx/error.log;

    # Performance
    keepalive_timeout 15;
    keepalive_requests 10000;

    client_body_buffer_size 1m;
    client_header_buffer_size 1k;
    large_client_header_buffers 4 8k;
    client_max_body_size 50m;

    send_timeout 10;
    client_body_timeout 10;
    client_header_timeout 10;

    # Gzip
    gzip on;
    gzip_disable "msie6";
    gzip_proxied any;
    gzip_comp_level 5;
    gzip_min_length 256;
    gzip_types
        text/plain
        text/css
        application/json
        application/javascript
        text/xml
        application/xml
        application/xml+rss
        image/svg+xml;

    # Proxy Cache Path
    proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mycache:100m inactive=60m;

    # Include Vhosts
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

### 🧪 Benchmark Report After Configuration Level Tunning (nginx.conf)
```
wrk -t2 -c10 -d10s http://192.168.20.126/
Running 10s test @ http://192.168.20.126/
  2 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   768.36us  561.46us  18.79ms   90.62%
    Req/Sec     6.74k     2.48k   10.68k    52.00%
  134135 requests in 10.00s, 110.14MB read
Requests/sec:  13412.26
Transfer/sec:     11.01MB
```
------------------------------------------------------------------------


## 🧠 2. Kernel Network Tuning (sysctl)

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

### 🧪 Benchmark Report After Kernel Level Tunning (sysctl)
```
wrk -t2 -c10 -d10s http://192.168.20.126/
Running 10s test @ http://192.168.20.126/
  2 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency     0.88ms  466.74us   4.68ms   74.12%
    Req/Sec     5.81k     1.66k    9.22k    75.50%
  115678 requests in 10.00s, 94.98MB read
Requests/sec:  11567.28
Transfer/sec:      9.50MB
```
------------------------------------------------------------------------

## 🌐 3. Enable HTTP/2

``` nginx
server {
    listen 443 ssl http2;
    ...
}
```

------------------------------------------------------------------------

## 🤝 4. Load Balancer Example

``` nginx
upstream backend {
    zone backend 64k;
    least_conn;
    server app01 weight=1;
    server app02 weight=1;
}
```

------------------------------------------------------------------------

## 🧹 5. Logging Optimization

``` nginx
access_log off;
```

or:

``` nginx
access_log /var/log/nginx/access.log main buffer=512k flush=1m;
```

------------------------------------------------------------------------

## 🧪 6. Benchmark Tools

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

## ▶️ 7. Test & Reload Nginx

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
