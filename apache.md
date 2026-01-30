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
