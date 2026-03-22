# 🚀 Nginx & Nginx Extras Installation Guide (Ubuntu & RHEL)

This document includes **both** standard Nginx installation instructions
**and** Nginx Extras installation instructions.

------------------------------------------------------------------------

# 🟩 PART 1 --- Standard Nginx Installation

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

Visit:

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

------------------------------------------------------------------------

# 🟦 PART 2 --- Nginx Extras Installation (Ubuntu & RHEL)

Nginx Extras provides many additional compiled modules including:

-   `headers-more`\
-   `auth-request`\
-   `echo`\
-   `lua` module\
-   `image-filter`\
-   `geoip`\
-   and more.

------------------------------------------------------------------------

# 🐧 Install Nginx Extras on Ubuntu (20.04/22.04/24.04)

### 🔄 Step 1: Update System Packages

``` bash
sudo apt update
sudo apt upgrade -y
```

### 📦 Step 2: Install Nginx Extras

``` bash
sudo apt install nginx-extras -y
```

📌 **NOTE:** Installing `nginx-extras` *replaces* the standard `nginx`
package.

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

Visit:

    http://your_server_ip

------------------------------------------------------------------------

# 🛡 Install Equivalent Extras on RHEL / CentOS / Rocky / AlmaLinux

⚠️ **There is no direct `nginx-extras` package** for RHEL-based
distributions.

Instead, install extra modules individually.

### 🔄 Step 1: Update System

``` bash
sudo dnf update -y
```

### 📦 Step 2: Enable EPEL Repository

``` bash
sudo dnf install epel-release -y
```

### 🌐 Step 3: Install Standard Nginx

``` bash
sudo dnf install nginx -y
```

### 📦 Step 4: Install Extra Modules

Examples:

``` bash
sudo dnf install nginx-mod-http-headers-more
sudo dnf install nginx-mod-http-image-filter
sudo dnf install nginx-mod-http-perl
sudo dnf install nginx-mod-stream
```

### ▶️ Step 5: Enable & Start Service

``` bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 🌍 Step 6: Verify Installation

Visit:

    http://your_server_ip

------------------------------------------------------------------------

# 🎉 You now have both Standard Nginx & Nginx Extras installation guides!
