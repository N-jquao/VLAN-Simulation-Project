# The following shows my process of setting up each service vm to make them available for use

1. On each vm carry out updates

```bash
sudo apt update && sudo apt upgrade -y
```
2. Create a sudo user and disable root SSH login in /etc/ssh/sshd_config
3. Set up SSH keys for passwordless, secure access
4. Set up the database server (backend)

 ```bash
 sudo apt install mariadb-server
 sudo mysql_secure_installation
 sudo vi /etc/mysql/mariadb.conf.d/50-server.cnf
 ```
  Change bind-address from 127.0.0.1 to 0.0.0.0 (to allow all to connect) or to the web server's ip address

 ```bash
 sudo mysql -u root -p
 CREATE USER 'webuser'@'<web-vm ip address>' IDENTIFIED BY '<your_strong_password>';
 CREATE DATABASE my_web_project;
 SHOW DATABASES;
 GRANT ALL PRIVILEGES ON <your_database_name>.* TO 'webuser'@'<web-vm ip address';
 FLUSH PRIVILEGES;
 EXIT;
```

5. Set up the file (storage) server for shared assets or backups
6. Install Samba on the web server vm

```bash
sudo apt install samba
sudo mkdir -p /srv/share
sudo chown nobody:nogroup /srv/share
sudo chmod 2775 /srv/share
```

7. Configure share - add the following to /etc/samba/smb.conf

[linux_share]
   comment = Project Shared Storage
   path = /srv/share
   browsable = yes
   read only = no
   guest ok = yes
   create mask = 0664
   directory mask = 0775

```bash
sudo systemctl restart smbd
```
8. On the web server mount the "/mnt" folder so it looks like a local folder

```bash
sudo apt install cifs-utils -y
sudo mkdir /mnt/network_files
sudo mount -t cifs //<ip address of file server>/linux_share /mnt/network_files -o guest
sudo vi /etc/fstab
```
   - Add the following to /etc/fstab on web server
   //<ip address of file server>/linux_share  /mnt/network_files  cifs  guest,uid=1000,gid=1000,iocharset=utf8  0  0

```bash
sudo  mount -a
df -h | grep linux_share
```

9. Set up the web server vm

```bash
sudo apt update
sudo apt install mariadb-client -y
mysql -u webuser -h <ip address of web vm” -p
sudo apt install nginx
```
   - App Logic: Install PHP or Python/Node.js depending on your stack.
   - Connectivity: In your application's configuration file (like wp-config.php or .env), point the database host to the 
     Database Server's Private IP.
   - SSL: Use Certbot (sudo apt install certbot python3-certbot-nginx) to get free Let's Encrypt certificates.

10. Set up the mail server
    - The most complex part. You will need a Fully Qualified Domain Name (FQDN).
    - MTA (Postfix): Handles sending/routing. Set it as an "Internet Site."
    - MDA (Dovecot): Handles delivery to mailboxes and IMAP/POP3 access.
    - DNS is Critical: You must set up MX Records pointing to this server’s public IP, along with SPF, DKIM, and DMARC
      records to prevent your emails from being marked as spam.

11. Final integration and firewall
    On each server, configure the firewall (UFW) to only allow what is necessary

    ```bash
    sudo ufw allow 80
    sudo ufw allow 443
    sudo ufw allow 22
    sudo ufw allow from <vm ip address> to any port 3306
    ```
12. Internal Sync: Mount the File Server shares onto the Web Server using /etc/fstab for persistent storage
