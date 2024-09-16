To use two different PHP versions (e.g., PHP 7.2 and PHP 8.3) for two different applications on Ubuntu 20.04, you can achieve this by configuring PHP-FPM for each version and specifying the desired version for each application using Apache's `SetHandler` or `ProxyPassMatch` directives.

Here’s a step-by-step guide to configure two PHP versions for two apps located at `/var/www/html/app1` and `/var/www/html/app2`:

### Step 1: Install PHP and PHP-FPM for Both Versions
Make sure both PHP 7.2 and PHP 8.3, along with their `php-fpm` versions, are installed:

```bash
sudo apt update
sudo apt install php7.2 php7.2-fpm php7.2-mbstring php7.2-xml php7.2-mysql
sudo apt install php8.3 php8.3-fpm php8.3-mbstring php8.3-xml php8.3-mysql
```

Start and enable `php-fpm` services for both versions:

```bash
sudo systemctl start php7.2-fpm
sudo systemctl enable php7.2-fpm

sudo systemctl start php8.3-fpm
sudo systemctl enable php8.3-fpm
```

### Step 2: Configure Apache to Use PHP-FPM
You need to configure Apache to route requests to the correct PHP-FPM service depending on the application.

#### 1. Enable Necessary Apache Modules:
```bash
sudo a2enmod proxy_fcgi setenvif
sudo a2enconf php7.2-fpm
sudo a2enconf php8.3-fpm
```

Restart Apache:

```bash
sudo systemctl restart apache2
```

#### 2. Configure Apache for Each Application
You can configure PHP-FPM using the `SetHandler` directive for each app's directory.

Open the Apache configuration file for your site. For example:

```bash
sudo nano /etc/apache2/sites-available/000-default.conf
```

Inside the configuration file, add the following blocks for each app:

```apache
# For App1 - Using PHP 7.2
<Directory /var/www/html/app1>
    AllowOverride All
    Require all granted
    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php7.2-fpm.sock|fcgi://localhost"
    </FilesMatch>
</Directory>

# For App2 - Using PHP 8.3
<Directory /var/www/html/app2>
    AllowOverride All
    Require all granted
    <FilesMatch \.php$>
        SetHandler "proxy:unix:/run/php/php8.3-fpm.sock|fcgi://localhost"
    </FilesMatch>
</Directory>
```

#### Explanation:
- The `SetHandler` directive is used to route `.php` files to the correct PHP-FPM version. 
- `proxy:unix:/run/php/php7.2-fpm.sock|fcgi://localhost` specifies that the requests are forwarded to the PHP 7.2 FPM socket for `app1` and to PHP 8.3 FPM for `app2`.

### Step 3: Restart Apache
After making these changes, restart Apache to apply the new configuration:

```bash
sudo systemctl restart apache2
```

### Step 4: Test the Setup
Now, visit the URLs for both applications:
- For **app1**, it should use PHP 7.2: `http://your-server-ip/app1`
- For **app2**, it should use PHP 8.3: `http://your-server-ip/app2`

To verify, you can create a simple `phpinfo()` page in each app’s directory:

```bash
echo "<?php phpinfo(); ?>" > /var/www/html/app1/info.php
echo "<?php phpinfo(); ?>" > /var/www/html/app2/info.php
```

Access `http://your-server-ip/app1/info.php` and `http://your-server-ip/app2/info.php` to check which PHP version is being used by each app.

### Conclusion
This setup allows you to run two different PHP versions on the same server for different applications without using virtual hosts, by configuring Apache to route PHP requests to the correct PHP-FPM version for each app.