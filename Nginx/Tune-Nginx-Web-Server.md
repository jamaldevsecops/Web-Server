# ⚡ Nginx High-Performance Tuning (Low Latency & High Traffic)

## 🧠 1. Optimize Worker Processes & Connections

Edit `/etc/nginx/nginx.conf`:

``` nginx
worker_processes auto;
worker_cpu_affinity auto;
worker_rlimit_nofile 200000;

events {
    worker_connections 65535;
    multi_accept on;
    use epoll;
}
```

------------------------------------------------------------------------

## 📁 2. Increase OS File Descriptor Limits

### Edit `/etc/security/limits.conf`:

    * soft nofile 200000
    * hard nofile 200000

### Create override file:

`/etc/systemd/system/nginx.service.d/override.conf`

``` ini
[Service]
LimitNOFILE=200000
```

Reload:

``` bash
systemctl daemon-reload
systemctl restart nginx
```

------------------------------------------------------------------------

## 🚀 3. Enable Sendfile & TCP Performance Options

Inside `http {}`:

``` nginx
sendfile on;
tcp_nopush on;
tcp_nodelay on;
keepalive_timeout 15;
keepalive_requests 10000;
```

------------------------------------------------------------------------

## 📦 4. Buffer & Timeout Settings

``` nginx
client_body_buffer_size 1m;
client_header_buffer_size 1k;
large_client_header_buffers 4 8k;
client_max_body_size 50m;

send_timeout 10;
client_body_timeout 10;
client_header_timeout 10;
```

------------------------------------------------------------------------

## 🗜 5. Enable Gzip Compression

``` nginx
gzip on;
gzip_disable "msie6";
gzip_comp_level 5;
gzip_min_length 256;
gzip_proxied any;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss image/svg+xml;
```

------------------------------------------------------------------------

## 🧊 6. Enable Static & Proxy Caching

### Static file caching:

``` nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 30d;
    access_log off;
}
```

### Proxy caching:

``` nginx
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=mycache:100m inactive=60m;
proxy_cache mycache;
proxy_cache_use_stale error timeout invalid_header updating;
```

------------------------------------------------------------------------

## ⚙️ 7. Linux Kernel Network Tuning

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

Apply:

``` bash
sysctl -p
```

------------------------------------------------------------------------

## 🌐 8. Enable HTTP/2

``` nginx
server {
    listen 443 ssl http2;
}
```

------------------------------------------------------------------------

## 🤝 9. Load Balancer Example

``` nginx
upstream backend {
    zone backend 64k;
    least_conn;
    server app01 weight=1;
    server app02 weight=1;
}
```

------------------------------------------------------------------------

## 📉 10. Logging Optimization

``` nginx
access_log off;
```

**or**

``` nginx
access_log /var/log/nginx/access.log main buffer=512k flush=1m;
```

------------------------------------------------------------------------

## 🧪 11. Benchmarking Tools

### 🚀 wrk

``` bash
wrk -t8 -c2000 -d30s http://yourserver/
```

### 📊 Apache Benchmark (ab)

``` bash
ab -n 100000 -c 2000 http://yourserver/
```

### 🛡 Siege

``` bash
siege -c 2000 -t 1M http://yourserver/
```
