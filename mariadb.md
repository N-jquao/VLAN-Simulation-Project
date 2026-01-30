# The following shows the process of installing MySQL/MariaDB on an ubuntu vm

```bash
# Install MariaDB
sudo apt install mariadb-server mariadb-client -y
sudo systemctl status mariadb
sudo systemctl enable mariadb

# Secure the installation as by default MariaDB has some insecure settings
sudo mysql_secure_installation
- press enter (there's no password yet)
- answer n (we'll use password auth)
- and Y (then enter a strong password)
- answer Y (security best practice to remove anonymous users)
- answer Y (security best practice to disallow root login)
- answer Y (remove test database as we don't need it)
- answer Y (to reload privilege tables and apply all changes)

# Log into MySQL
sudo mysql -u root -p # -u root (login as root user), -p (prompt for password)
# Enter the password you set

# Create a database for the test website
CREATE DATABASE testsite_db;
SHOW DATABASES;

# Create a database user
# Never use the root account for applications! Create a dedicated user
CREATE USER 'testsite_user'@'localhost' IDENTIFIED BY '<SecurePassword>';
# What this does
- creates user testsite_user
- @'localhost' = can only connect from this server (secure)

# Grant permissions
# Give the user access to the database
GRANT ALL PRIVILEGES ON testsite_db.* TO 'testsite_user'@'localhost';
# What this does
- GRANT ALL PRIVILEGES = full access
- testsite_db.* = to all tables in testsite_db
- TO 'testsite_user'@'localhost' = for this specific user

# Apply the changes
FLUSH PRIVILEGES;

# Exit MySQL and test the new user
EXIT;
mysql -u testsite_user -p testsite_db
# Enter the password you set for testsite_user

# Try creating a test table
CREATE TABLE test (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

# Check it worked
SHOW TABLES;

# Insert some test data


