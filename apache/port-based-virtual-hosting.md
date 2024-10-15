# Port Based Virtual Hosting

To run Apache on both port 80 and 8081 with IP-based virtual hosting, and to serve your web application located in /var/www/html/app1, follow these steps:

1. Enable Apache to Listen on Both Ports

First, configure Apache to listen on both ports 80 and 8081.

Edit the Apache ports configuration file:

sudo nano /etc/apache2/ports.conf

Add the following lines (if they don’t already exist):

Listen 80
Listen 8081

2. Create the Virtual Host Configuration

Now, create a virtual host file for the IP-based virtual host on port 8081.

Create a new virtual host configuration file:

sudo nano /etc/apache2/sites-available/app1-8081.conf

Add the following configuration:

<VirtualHost *:8081>
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/app1

    <Directory /var/www/html/app1>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

This sets Apache to serve content from /var/www/html/app1 when accessed via http://your_server_ip:8081.

3. Enable the New Virtual Host

Enable the virtual host configuration:

sudo a2ensite app1-8081.conf

4. Restart Apache

Restart Apache to apply the changes:

sudo systemctl restart apache2

5. Check Firewall (Optional)

If you have a firewall enabled, ensure that port 8081 is open. For example, with UFW (Uncomplicated Firewall), you can run:

sudo ufw allow 8081/tcp

6. Access the Web Application

Now, you should be able to access your web application by navigating to:

http://your_server_ip:8081

This will serve the content from /var/www/html/app1 on port 8081, while Apache continues to serve other content on port 80 as usual.
