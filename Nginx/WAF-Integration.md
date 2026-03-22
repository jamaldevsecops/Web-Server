# 🔐 WAF Integration Runbook (ModSecurity + OWASP CRS) for Nginx

---

## 📌 Overview
This runbook provides **production-grade integration of ModSecurity (WAF)** with **Nginx** using **OWASP Core Rule Set (CRS)**.

---

# 🧰 Prerequisites
- Nginx installed
- Root/sudo access
- Internet access for package installation

---

# 🟢 1. Install Dependencies

```bash
sudo dnf install -y gcc make git wget curl pcre-devel zlib-devel openssl-devel libxml2-devel yajl-devel
```

---

# 🟢 2. Install ModSecurity

```bash
cd /opt
sudo git clone https://github.com/SpiderLabs/ModSecurity
cd ModSecurity
sudo git submodule init
sudo git submodule update
sudo ./build.sh
sudo ./configure
sudo make
sudo make install
```

---

# 🟢 3. Install ModSecurity-nginx Connector

```bash
cd /opt
sudo git clone https://github.com/SpiderLabs/ModSecurity-nginx.git
```

---

# 🟢 4. Rebuild Nginx with ModSecurity

```bash
nginx -V
```

Recompile Nginx with:
```bash
--add-module=/opt/ModSecurity-nginx
```

---

# 🟢 5. Configure ModSecurity

```bash
sudo mkdir -p /etc/nginx/modsec
sudo cp /opt/ModSecurity/modsecurity.conf-recommended /etc/nginx/modsec/modsecurity.conf
sudo cp /opt/ModSecurity/unicode.mapping /etc/nginx/modsec/
```

Edit:
```bash
sudo nano /etc/nginx/modsec/modsecurity.conf
```

Change:
```ini
SecRuleEngine On
```

---

# 🟢 6. Install OWASP CRS

```bash
cd /etc/nginx/modsec
sudo git clone https://github.com/coreruleset/coreruleset.git
cd coreruleset
sudo cp crs-setup.conf.example crs-setup.conf
```

---

# 🟢 7. Create ModSecurity Include File

```bash
sudo nano /etc/nginx/modsec/main.conf
```

```apache
Include /etc/nginx/modsec/modsecurity.conf
Include /etc/nginx/modsec/coreruleset/crs-setup.conf
Include /etc/nginx/modsec/coreruleset/rules/*.conf
```

---

# 🟢 8. Enable ModSecurity in Nginx

Edit Nginx config:

```nginx
load_module modules/ngx_http_modsecurity_module.so;

http {
    modsecurity on;
    modsecurity_rules_file /etc/nginx/modsec/main.conf;
}
```

---

# 🧪 9. Test WAF

```bash
sudo nginx -t
sudo systemctl restart nginx
```

Test attack:
```bash
curl "http://web1.ngd.com/?id=' OR 1=1 --"
```

Expected: Blocked request

---

# 🔒 10. Tuning (Production)

## Detection Only Mode (Initial)
```ini
SecRuleEngine DetectionOnly
```

## Then enable blocking
```ini
SecRuleEngine On
```

---

# 📊 11. Logging

```ini
SecAuditLog /var/log/modsec_audit.log
SecAuditLogParts ABIJDEFHZ
```

---

# 🚀 12. Performance Optimization

- Disable unnecessary CRS rules
- Use anomaly scoring
- Tune request body limits:
```ini
SecRequestBodyLimit 13107200
```

---

# 🛡️ 13. Best Practices

- Start in detection mode
- Monitor logs
- Whitelist false positives
- Regularly update CRS

---

# 🧯 14. Troubleshooting

## Check logs
```bash
tail -f /var/log/nginx/error.log
tail -f /var/log/modsec_audit.log
```

## Common Issues
- Module not loaded → check `nginx -V`
- False positives → adjust CRS rules

---

# ✅ Final Checklist

- [ ] ModSecurity installed
- [ ] CRS installed
- [ ] Nginx module enabled
- [ ] WAF blocking tested
- [ ] Logs monitored

---

# 🎯 Conclusion
You now have a **production-grade WAF (ModSecurity + OWASP CRS)** protecting your Nginx server.
