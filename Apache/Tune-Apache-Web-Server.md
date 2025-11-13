# ⚡ Apache High-Performance Tuning Guide

### Handle High Traffic & Reduce Latency

------------------------------------------------------------------------

# 🧠 1. Choose the Right Apache MPM Module

Check active MPM:

``` bash
apache2ctl -V | grep MPM
```

### Recommended: **event MPM**

Ubuntu:

``` bash
sudo a2dismod mpm_prefork
sudo a2enmod mpm_event
```

RHEL:

``` bash
sudo sed -i 's/prefork/event/' /etc/httpd/conf.modules.d/00-mpm.conf
```

------------------------------------------------------------------------

# ⚙️ 2. Tune the event MPM Settings

Edit:

Ubuntu: `/etc/apache2/mods-available/mpm_event.conf`\
RHEL: `/etc/httpd/conf.modules.d/00-mpm.conf`

``` apache
<IfModule mpm_event_module>
    StartServers              4
    ServerLimit               1024
    MaxRequestWorkers         1024
    MinSpareThreads           75
    MaxSpareThreads           250
    ThreadLimit               64
    ThreadsPerChild           25
</IfModule>
```

------------------------------------------------------------------------

# 🚀 3. Enable HTTP/2

Ubuntu:

``` bash
sudo a2enmod http2
sudo systemctl restart apache2
```

RHEL:

``` bash
sudo dnf install mod_http2
sudo systemctl restart httpd
```

Add to VirtualHost:

``` apache
Protocols h2 h2c http/1.1
```

------------------------------------------------------------------------

# 📦 4. Enable Caching

``` bash
sudo a2enmod cache cache_disk expires headers
```

Example:

``` apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresDefault "access plus 1 month"
</IfModule>
```

------------------------------------------------------------------------

# 🚄 5. Enable Gzip Compression

Enable:

``` bash
sudo a2enmod deflate
```

Add:

``` apache
AddOutputFilterByType DEFLATE text/plain text/html text/css application/json application/javascript text/xml image/svg+xml
```

------------------------------------------------------------------------

# 📁 6. File Descriptor Limits

``` bash
echo "* hard nofile 200000" | sudo tee -a /etc/security/limits.conf
echo "* soft nofile 200000" | sudo tee -a /etc/security/limits.conf
```

------------------------------------------------------------------------

# ⚙️ 7. Kernel Network Tuning

Add to `/etc/sysctl.conf`:

``` bash
net.core.somaxconn = 65535
net.ipv4.ip_local_port_range = 1024 65000
net.ipv4.tcp_fin_timeout = 15
net.core.netdev_max_backlog = 65535
```

Apply:

``` bash
sudo sysctl -p
```

------------------------------------------------------------------------

# 🧪 8. Benchmark Tools

### wrk

``` bash
wrk -t8 -c2000 -d30s http://yourserver/
```

### ab

``` bash
ab -n 100000 -c 2000 http://yourserver/
```

### siege

``` bash
siege -c 2000 -t 1M http://yourserver/
```
