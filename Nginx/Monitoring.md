# 📊 Monitoring & Compliance Dashboard Runbook  
## (Prometheus + Grafana + Logs for Nginx + PHP + WAF)

---

## 📌 Overview
This runbook covers:
- 📈 Prometheus monitoring
- 📊 Grafana dashboards
- 🧾 Log aggregation
- 🔐 Compliance tracking visualization

---

# 🟢 1. Architecture Overview

```
Nginx / PHP / WAF
        ↓
   Exporters (Metrics)
        ↓
   Prometheus
        ↓
   Grafana Dashboards
        ↓
   Alerts / Compliance View
```

---

# 📦 2. Install Prometheus

```bash
sudo useradd --no-create-home --shell /bin/false prometheus

cd /opt
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-linux-amd64.tar.gz
tar -xvf prometheus-linux-amd64.tar.gz
sudo mv prometheus-* prometheus
```

### Create config:
```bash
sudo nano /opt/prometheus/prometheus.yml
```

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'nginx'
    static_configs:
      - targets: ['localhost:9113']

  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
```

---

# 🟢 3. Install Exporters

## Node Exporter
```bash
sudo dnf install -y node_exporter
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

## Nginx Exporter
```bash
docker run -d -p 9113:9113 nginx/nginx-prometheus-exporter
```

---

# 🟢 4. Enable Nginx Metrics

```nginx
location /nginx_status {
    stub_status;
    allow 127.0.0.1;
    deny all;
}
```

---

# 🟢 5. Install Grafana

```bash
sudo dnf install -y https://dl.grafana.com/oss/release/grafana-10.x.rpm
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

Access:
👉 http://SERVER_IP:3000

---

# 🟢 6. Add Prometheus Data Source

- Go to **Grafana → Settings → Data Sources**
- Add:
  - URL: `http://localhost:9090`

---

# 📊 7. Dashboard Metrics (Production)

## Nginx Metrics
- Requests/sec
- Active connections
- 4xx / 5xx errors

## System Metrics
- CPU usage
- Memory usage
- Disk I/O

## PHP-FPM
- Active processes
- Slow requests

---

# 🔐 8. Compliance Dashboard (CIS Mapping)

## Example Panels

### 🔐 TLS Compliance
- TLS version usage
- Weak cipher detection

### 🛡️ Security Headers
- Missing headers count

### 🚫 Failed Requests
- WAF blocked requests

### 📜 Logging
- Log volume
- Error spikes

---

# 📊 Example PromQL Queries

## Nginx Requests
```promql
rate(nginx_http_requests_total[1m])
```

## Error Rate
```promql
rate(nginx_http_requests_total{status=~"5.."}[5m])
```

## CPU Usage
```promql
100 - (avg by(instance)(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

---

# 🧾 9. Log Aggregation (Recommended)

## Install Loki
```bash
docker run -d -p 3100:3100 grafana/loki
```

## Install Promtail
```bash
docker run -d   -v /var/log:/var/log   grafana/promtail
```

---

# 📊 10. Grafana Log Dashboard

- Filter logs:
  - Nginx errors
  - ModSecurity alerts
- Create panels:
  - Top attack IPs
  - Blocked requests

---

# 🚨 11. Alerts (Production)

## Example Alert Rule
- High error rate (>5%)
- CPU > 80%
- WAF blocks spike

---

# ⚙️ 12. Compliance Tracking Dashboard Design

## Sections:
1. 🔐 Security Status
2. 📊 Performance Health
3. 🚨 Incident Alerts
4. 📜 Audit Logs

---

# 🧠 Best Practices

✔ Use **labels for services**  
✔ Separate dashboards per environment  
✔ Enable **alerting rules**  
✔ Store logs centrally  

---

# ✅ Final Checklist

- [ ] Prometheus running
- [ ] Exporters working
- [ ] Grafana dashboards created
- [ ] Logs integrated (Loki)
- [ ] Alerts configured

---

# 🎯 Conclusion
You now have:
- 📈 Full observability
- 🔐 Compliance visibility
- 🚨 Real-time alerting

