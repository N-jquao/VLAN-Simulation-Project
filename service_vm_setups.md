# The following shows my process of setting up each service vm to make them available for use

```bash

```

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


