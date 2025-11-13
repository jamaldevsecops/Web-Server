# 🚀 Nginx Installation Guide (Ubuntu & RHEL)

## 🐧 Install Nginx on Ubuntu (20.04/22.04/24.04)

### 🔄 Step 1: Update System Packages

``` bash
sudo apt update
sudo apt upgrade -y
```

### 📦 Step 2: Install Nginx

``` bash
sudo apt install nginx -y
```

### ▶️ Step 3: Enable & Start Nginx

``` bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 🩺 Step 4: Check Service Status

``` bash
sudo systemctl status nginx
```

### 🔥 Step 5: Allow HTTP/HTTPS Through Firewall

``` bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
```

### 🌍 Step 6: Verify Installation

Open your browser and visit:

    http://your_server_ip

------------------------------------------------------------------------

## 🛡 Install Nginx on RHEL / CentOS / Rocky / AlmaLinux

### 🔄 Step 1: Update System

``` bash
sudo dnf update -y
```

### 📦 Step 2: Install EPEL Repository

``` bash
sudo dnf install epel-release -y
```

### 🌐 Step 3: Install Nginx

``` bash
sudo dnf install nginx -y
```

### ▶️ Step 4: Enable & Start Service

``` bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 🔥 Step 5: Allow Firewall Traffic

``` bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 🌍 Step 6: Verify Installation

Visit:

    http://your_server_ip
