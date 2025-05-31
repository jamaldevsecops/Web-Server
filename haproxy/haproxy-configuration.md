# 🛡️ HAProxy Configuration Notes – Ubuntu 20.04
![image](https://github.com/user-attachments/assets/fca7b783-9c1b-47d9-95de-a7ff281ead53)

## 📘 Scenario Overview
This HAProxy setup is designed to function as a reverse proxy and load balancer for two internal business applications:
- Wingssfa (Sales Force Automation) – accessed via https://my.wingssfa.net
- Wingshrms (Human Resource Management System) – accessed via https://my.wingshrms.net

Both applications run on two backend servers each (in the 192.168.20.0/24 and 192.168.10.0/24 subnets), and HAProxy performs SSL termination and intelligent routing based on SNI (Server Name Indication).

🌐 Architecture Overview
```markdown
               ┌──────────────────────────────┐
               │      Internet / Clients      │
               └────────────┬─────────────────┘
                            │
                            ▼
                ┌───────────────────────────┐
                │       HAProxy (443)       │
                │  my.wingssfa.net / HRMS   │
                └──────┬─────────┬──────────┘
                       │         │
            ┌──────────▼──┐   ┌──▼──────────┐
            │ Wings SFA  │   │ Wings HRMS  │
            │ Backends   │   │ Backends    │
            └────────────┘   └─────────────┘
```
## ✅ Installation on Ubuntu 20.04
```bash
sudo apt update
sudo apt install -y haproxy
```

Enable and start the service:
```bash
sudo systemctl enable haproxy
sudo systemctl start haproxy
```
## 🔐 SSL Configuration
1. ### SSL Certificates
Place your SSL certificate bundle in:
```bash
/etc/ssl/certs/all/
```
Ensure HAProxy has a valid PEM file for each domain:
```bash
# Example (concatenated cert + key)
sudo cat wingssfa.net.crt CA_Bundle.crt wingssfa.net.key > wingssfa.net.pem
sudo chown :haproxy /etc/ssl/certs/all/wingssfa.net.pem
```

2. ### Generate Diffie-Hellman Parameters
```bash
sudo curl https://ssl-config.mozilla.org/ffdhe2048.txt -o /etc/ssl/certs/dhparam.pem
```
## 🧠 Backend Configuration Summary
## 🔁 sfa_backend – Wings SFA
```haproxy
backend sfa_backend
    mode http
    balance roundrobin
    cookie SFAID insert indirect nocache
    server sfa1 192.168.20.254:81 cookie sfa1 check inter 5s fall 3 rise 2 slowstart 10s maxconn 100
    server sfa2 192.168.10.254:81 cookie sfa2 check inter 5s fall 3 rise 2 slowstart 10s maxconn 100
```
- Sticky sessions with cookie SFAID
- Load balanced in roundrobin fashion

## 🔁 hrms_backend – Wings HRMS
```haproxy
backend hrms_backend
    mode http
    balance roundrobin
    cookie HRMSID insert indirect nocache
    server hrms1 192.168.20.254:5380 cookie hrms1 check inter 5s fall 3 rise 2 slowstart 10s maxconn 100
    server hrms2 192.168.10.254:5380 cookie hrms2 check inter 5s fall 3 rise 2 slowstart 10s maxconn 100
```
- Sticky sessions with cookie HRMSID

## ❌ default_backend
```haproxy
backend default_backend
    mode http
    http-request deny deny_status 400
```
- Rejects unknown requests (e.g., wrong SNI)

## 🔍 Frontend Logic – SNI-based Routing
```haproxy
frontend https_front
    bind *:443 ssl crt /etc/ssl/certs/all/ alpn h2,http/1.1
    mode http

    http-response set-header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload"
    http-response set-header X-Content-Type-Options nosniff
    http-response set-header X-Frame-Options DENY
    http-response set-header Content-Security-Policy "default-src 'self';"

    acl is_sfa ssl_fc_sni -i my.wingssfa.net
    acl is_hrms ssl_fc_sni -i my.wingshrms.net

    use_backend sfa_backend if is_sfa
    use_backend hrms_backend if is_hrms
    default_backend default_backend
```

## ⚙️ Global SSL & Security Tuning
```haproxy
ssl-default-bind-curves X25519:prime256v1:secp384r1
ssl-default-bind-ciphers ECDHE-...:DHE-...
ssl-default-bind-ciphersuites TLS_AES_...
ssl-default-bind-options prefer-client-ciphers ssl-min-ver TLSv1.2 no-tls-tickets

ssl-dh-param-file /etc/ssl/certs/dhparam.pem
```
- Enforces modern cipher suites and TLS 1.2+
- Sets secure Diffie-Hellman parameters

## 🧪 Syntax Check Before Restart
Always validate configuration before reloading:
```bash
sudo haproxy -c -f /etc/haproxy/haproxy.cfg
```

## 📊 HAProxy Stats Page
***Configuration***
```haproxy
listen stats
    bind *:8404
    acl allowed_stats src 192.168.20.0/24 192.168.10.0/24
    #http-request deny unless allowed_stats
    stats enable
    stats uri /stats
    stats refresh 10s
    stats auth admin:YourStrongPassword
```
***Access***
Open in browser:
```plain
http://<haproxy-ip>:8404/stats
```
Login using:
- Username: admin
- Password: YourStrongPassword
![image](https://github.com/user-attachments/assets/1c860655-d9c9-4993-bb05-672295c6cd46)


## 🔁 Restart HAProxy After Changes
```bash
sudo systemctl restart haproxy
```
