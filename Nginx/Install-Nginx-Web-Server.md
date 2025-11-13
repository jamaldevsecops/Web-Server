# Install Nginx on Ubuntu and RHEL

## Install Nginx on Ubuntu (20.04/22.04/24.04)

### 1. Update system packages

``` bash
sudo apt update
sudo apt upgrade -y
```

### 2. Install Nginx

``` bash
sudo apt install nginx -y
```

### 3. Enable and start Nginx

``` bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 4. Check service status

``` bash
sudo systemctl status nginx
```

### 5. Allow HTTP/HTTPS through UFW

``` bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
```

### 6. Verify installation

Visit:

    http://your_server_ip

------------------------------------------------------------------------

## Install Nginx on RHEL / CentOS / Rocky / AlmaLinux

### 1. Update system

``` bash
sudo dnf update -y
```

### 2. Install EPEL repo

``` bash
sudo dnf install epel-release -y
```

### 3. Install Nginx

``` bash
sudo dnf install nginx -y
```

### 4. Enable and start service

``` bash
sudo systemctl enable nginx
sudo systemctl start nginx
```

### 5. Allow Nginx in firewalld

``` bash
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```

### 6. Verify installation

Visit:

    http://your_server_ip
