# 🚀 Apache Web Server Installation Guide (Ubuntu & RHEL)

## 🐧 Install Apache on Ubuntu (20.04 / 22.04 / 24.04)

### 🔄 Step 1: Update System

``` bash
sudo apt update
sudo apt upgrade -y
```

### 📦 Step 2: Install Apache2

``` bash
sudo apt install apache2 -y
```

### ▶️ Step 3: Enable & Start Service

``` bash
sudo systemctl enable apache2
sudo systemctl start apache2
```

### 🩺 Step 4: Check Status

``` bash
sudo systemctl status apache2
```

### 🔥 Step 5: Allow Firewall

``` bash
sudo ufw allow 'Apache Full'
sudo ufw reload
```

### 🌍 Step 6: Verify

Visit:

    http://your_server_ip

------------------------------------------------------------------------

## 🛡 Install Apache on RHEL / CentOS / Rocky / AlmaLinux

### 🔄 Step 1: Update System

``` bash
sudo dnf update -y
```

### 📦 Step 2: Install Apache (httpd)

``` bash
sudo dnf install httpd -y
```

### ▶️ Step 3: Enable & Start

``` bash
sudo systemctl enable httpd
sudo systemctl start httpd
```

### 🔥 Step 4: Allow Traffic

``` bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 🌍 Step 5: Verify

Visit:

    http://your_server_ip
