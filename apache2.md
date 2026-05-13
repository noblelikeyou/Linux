================================================================
 APACHE2 COMMAND REFERENCE — LAB 7 & LAB 8
 MIT450 – Linux 2
================================================================

----------------------------------------------------------------
 INSTALLATION
----------------------------------------------------------------
sudo apt install apache2 -y
sudo apt install php libapache2-mod-php php-cli php-mysql php-xml php-curl php-mbstring -y
sudo apt install mariadb-server -y
sudo apt install certbot python3-certbot-apache -y
sudo apt install lynx -y

----------------------------------------------------------------
 SERVICE MANAGEMENT (systemctl)
----------------------------------------------------------------
sudo systemctl start apache2
sudo systemctl enable apache2
sudo systemctl stop apache2
sudo systemctl restart apache2
sudo systemctl reload apache2
sudo systemctl status apache2

sudo systemctl start mariadb
sudo systemctl enable mariadb

----------------------------------------------------------------
 APACHE MODULES
----------------------------------------------------------------
sudo a2enmod ssl                        # Enable SSL module
sudo a2ensite example1.conf             # Enable a virtual host
sudo a2dissite example1.conf            # Disable a virtual host
sudo apache2ctl configtest              # Check config syntax (expect: Syntax OK)

----------------------------------------------------------------
 FIREWALL
----------------------------------------------------------------
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
sudo firewall-cmd --list-all

----------------------------------------------------------------
 DIRECTORY SETUP
----------------------------------------------------------------
sudo mkdir -p /var/www/example1/html
sudo chown -R $USER:$USER /var/www/example1/html
sudo chown -R www-data:www-data /var/www
sudo chmod -R 755 /var/www

----------------------------------------------------------------
 VIRTUAL HOST TEMPLATE — HTTP (Port 80)
----------------------------------------------------------------
# File location: /etc/apache2/sites-available/yoursite.conf

<VirtualHost *:80>
    ServerName yoursite.com
    ServerAlias www.yoursite.com
    Redirect permanent / https://yoursite.com/
    DocumentRoot /var/www/yoursite.com

    <Directory /var/www/yoursite.com>
        Options FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog /var/log/apache2/yoursite_error.log
    CustomLog /var/log/apache2/yoursite_access.log combined
</VirtualHost>

----------------------------------------------------------------
 VIRTUAL HOST TEMPLATE — HTTPS (Port 443)
----------------------------------------------------------------
<VirtualHost *:443>
    ServerName yoursite.com
    ServerAlias www.yoursite.com
    DocumentRoot /var/www/yoursite.com
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
    SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
</VirtualHost>

----------------------------------------------------------------
 DNS — /etc/hosts
----------------------------------------------------------------
# Open hosts file:
sudo vim /etc/hosts

# Example entries:
10.0.2.15   example1.yourname.ca
10.0.2.15   mine.com www.mine.com
10.0.2.15   minephp.com www.minephp.com

# Verify DNS:
getent hosts example1.yourname.ca
ping -c 1 example1.yourname.ca
dig example1.yourname.ca @localhost

----------------------------------------------------------------
 TESTING
----------------------------------------------------------------
sudo apache2ctl configtest             # Syntax check
curl http://example1.yourname.ca       # Test HTTP response
lynx http://mine.com                   # Browser test (q then y to quit)
lynx https://mine.com                  # Test HTTPS

----------------------------------------------------------------
 CERTBOT (SSL Certificates)
----------------------------------------------------------------
sudo certbot --apache -d mine.com -d www.mine.com
sudo certbot --apache -d minephp.com -d www.minephp.com

----------------------------------------------------------------
 MARIADB — DATABASE SETUP
----------------------------------------------------------------
sudo mysql

# Inside MariaDB:
CREATE DATABASE mine;
CREATE USER 'mineuser'@'localhost' IDENTIFIED BY 'StrongPassword1!';
GRANT ALL PRIVILEGES ON mine.* TO 'mineuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;

----------------------------------------------------------------
 WORDPRESS SETUP
----------------------------------------------------------------
cd /tmp
wget https://wordpress.org/latest.tar.gz
tar -xvf latest.tar.gz
sudo cp -r wordpress/* /var/www/mine.com/
sudo chown -R www-data:www-data /var/www/mine.com

# Configure wp-config.php:
cd /var/www/mine.com
sudo cp wp-config-sample.php wp-config.php
sudo vim wp-config.php
# Update: DB_NAME, DB_USER, DB_PASSWORD

----------------------------------------------------------------
 VIM QUICK REFERENCE
----------------------------------------------------------------
vim filename          # Open file
i                     # Enter insert mode
Esc                   # Exit insert mode
:w                    # Save
:wq                   # Save and quit
:q!                   # Quit without saving
:wq!                  # Force save and quit (read-only override)

================================================================
