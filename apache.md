# The following shows my process of setting up apache service on an ubuntu vm

```Bash
sudo apt update
sudo apt upgrade -y
# This refreshes the package list from repositories

# If you don't have one already create a non-root user with sudo privileges for administrative tasks
sudo adduser -m sysadmin
sudo usermod -aG sudo sysadmin

# Set up ufw
sudo ufw status
sudo ufw allow openssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
sudo ufw status verbose

# Install the apache web server package
sudo apt install apache2 -y
sudo systemctl status apache2
sudo systemctl enable apache2 # Should see active (running) in green and enabled

# Test it works in your browser
# Go to http://<server ip>
# You should see the application default page


# View the apache directory structure
ls -la /etc/apache2/

# Key directories
/etc/apache2/ --> main config dir
/etc/apache2/sites-available/ --> virtual host configs (disabled)
/etc/apache2/sites-enabled/ --> active virtual host configs (symlinks)
/var/www/html --> default web root (where website files go)
/var/log/apache2/ --> apache log files

# Create your first website
sudo mkdir -p /var/www/testsite.local
sudo vi /var/www/testsite.local/index.html
# Paste this content
<!DOCTYPE html>
<html>
<head>
    <title>My Test Site</title>
</head>
<body>
    <h1>Welcome to my LAMP Stack!</h1>
    <p>This is my first virtual host.</p>
    <p>Server: <?php echo gethostname(); ?></p>
</body>
</html>
# Save and exit

# Set ownership for www-data
# Change ownership to www-data (Apache's user)
sudo chown -R www-data:www-data /var/www/testsite.local
# Set appropriate perms
sudo chmod -R 755 /var/www/testsite.local
# Ensures Apache can read the files but other's can't modify them

# Create a new virtual host config file
sudo vi /etc/apache2/sites-available/testsite.local.conf
# Paste this configuration
<VirtualHost *:80>
    ServerAdmin admin@testsite.local
    ServerName testsite.local
    ServerAlias www.testsite.local
    DocumentRoot /var/www/testsite.local

    <Directory /var/www/testsite.local>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/testsite.local-error.log
    CustomLog ${APACHE_LOG_DIR}/testsite.local-access.log combined
</Virtualization>
# Save and exit
# This tells Apache how to serve this specific website

# Enable the new site
sudo a2ensite testsite.local.conf

# Test Apache config for syntax errors
sudo apache2ctl configtest # Expected output: "Syntax OK"

# Reload Apache to apply changes
sudo systemctl reload apache2

# For local testing, add an entry to your hosts file
sudo vi /etc/hosts
# Add this line
127.0.0.1 testsite.local
# Now test
curl http://testsite.local
# Should see your html

## Have completed Basic server setup, firewall config, Apache installation and co fig, and first virtual host creation

# From your host machine's browser go to
http://<vm ip address>
```
