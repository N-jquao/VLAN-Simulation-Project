#The following shows my process of using a VMnet (NAT) for the internet connection for my management server, and then using it as a router for 4 simulated VLANS, which are simulated by LAN segments.

1. Create 5 vms on VMware Workstation with Linux Ubuntu LTS OS on Windows 11 and name them respectively as stated above
2. Open Virtual Network Editor on Windows - ensure all vms have been powered off before
3. Click on change settings
4. Click on Add Network
5. Select VMnet1 and click ok
6. Select NAT option
7. Click apply then ok
8. Go to VMware Workstation and edit vm settings
9. Choose NAT and save
10. Power on the management server
11. Ping 8.8.8.8 or google.com to see if the internet works. It most probably won't work as the NAT config isn't complete
12. In Windows, open Network Connections
13. Right click on Wi-Fi and click properties
14. Click "sharing" at top and select "Allow other network users to connect through this computer's connection"
15. In dropdown select the VMNet1 you made
16. Deselect the bottom checkbox and click ok
17. Turn off the management vm
18. Reopen management vm's settings and click "custom network" under network settings, then click ok
19. Add a new Network adapter under network settings on the management server vm
20. Select "LAN segments" and click add. Name it "Internal_Lab"
21. Select the "Internal_Lab" option from the dropdown and save. This will be your second ethernet device on the vm (ens37 - it will act as the router for the service vms)
22. Start vm and try pinging 8.8.8.8. If it doesn't work, you may need to set up the ethernet device settings. Will get the IP info from Virtual Network settings

```bash
ip a
sudo ip link set ens33 up
sudo ip addr add 192.168.158.128/24 dev ens33
sudo ip route add default via 192.168.159.2
sudo ip link set ens37 up
sudo ip addr add 192.168.10.1 dev ens37
sudo resolvectl dns ens33 8.8.8.8 8.8.4.4
resolvectl status ens33
sudo resolvectl dns ens37 8.8.8.8 8.8.4.4
resolvectl status ens37
ip a
ping 8.8.8.8
ping 192.168.159.2
ping 192.168.159.128
ping 192.168.10.1
```
20. The management server pings should have worked - the management server is now ready and working
21. Now to set up SSH from Windows Command Prompt into the management server
22. Install openssh-server on both the mgmt server and Windows machine's command prompt
23. Get the ip address for the mgmt vm
24. Open Virtual Network Editor on Windows
25. Click "change settings" at bottom right
26. Select VMnet1
27. Click the NAT settings button
28. In the NAT settings window click add (under the port forwarding section)
29. Fill in the following details
    - Host port: 2222 (This is the port you will type into your ssh command on the windows command prompt
    - Type: TCP
    - Virtual machine ip address (Enter the Ubuntu ip you found in step 1 from Virtual Network editor): 192.168.159.128
    - Virtual machine port: 22 (Standard SSH port)
    - Description: SSH to Ubuntu vm

30. Click ok on all windows to save and apply the changes
31. Start the mgmt vm
32. Ensure the Ubuntu firewall allows traffic on port 22 and also port 2222

    ``` bash
    sudo ufw allow 22
    ```
  33. Press the Windows Key, type "Windows Defender Firewall with Advanced Security", and press Enter
  34. In the left sidebar click on inbound rules
  35. In the right sidebar click new rule
  36. Rule type - select port and click next
  37. Protocol and ports - select TCP
  38. Select specific local ports and enter 2222
  39. Action - select "Allow the connection"
  40. Profile - Keep Domain, Private, and Public checked (or just Private if you're on a trusted home network)
  41. Name - give it a clear name like "VMware SSH Forwarding (Port 2222)" and click finish
  42. Connect via windows cmd prompt - ssh linux@127.0.0.1 -p 2222
  43. Windows cmd prompt - ssh-keygen
  44. Enter following commands on mgmt vm

  ```bash
  sudo ssh-keygen
  ```
  46. Copy public key (.pub) from windows to authorized_keys file on vm

  ```bash
  sudo vi ~/.ssh/authorized_keys
  ```
  47. Add public key from windows to the authorized_keys file and save
  48. Open the ssh config file

  ```bash
  sudo vi /etc/ssh/sshd_config
  ```
  49.  Change "PasswordAuthentication" to "no" and "PubkeyAuthentication" to "yes"
  50.  Restart ssh

  ```bash
  sudo systemctl restart ssh
  sudo systemctl enable ssh
  sudo systemctl start ssh
  sudo systemctl status ssh
  ```
  51. Make a config.txt file in windows under C:\Users\username\.ssh\
  52. Put this is in the config file
      	Host myubuntu
          HostName 127.0.0.1
          User linux
          Port 2222
          IdentityFile ~/.ssh/id_ed25519
  53. Enter this in command prompt
        	cd C:\Users\najqu\.ssh
          ren config.txt config
  54. Login again on cmd prompt
	        ssh myubuntu
  ```bash
  sudo iptables -t nat -A POSTROUTING  -o ens33 -j MASQUERADE
  sudo iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
  sudo iptables -A FORWARD -j ACCEPT
  sudo netfilter-persistent save
  cat /proc/sys/net/ipv4/ip_forward #should be 1
  sudo vi /etc/netplan/50-cloud-init.yaml
  ```
  <img width="352" height="143" alt="image" src="https://github.com/user-attachments/assets/cbd61d3d-66cb-472d-9e57-4e571edd7393" />

  ```bash
  sudo netplan apply
  sudo reboot
  ```
55. Test the pings work
    
57. To set up the web server database, open the vm's network settings and select "LAN Segment"
58. Choose the "Internal_Lab" option from the dropdown and save
59. Start vm and try pinging - it should not work as ip configs have not been done

```bash
ip a
sudo ip link set ens33 up
sudo ip addr add 192.168.10.10/24 dev ens33
sudo ip route add default via 192.168.10.1
sudo resolvectl dns ens33 8.8.8.8 8.8.4.4
resolvectl status ens33
ping 8.8.8.8
ping 192.168.159.128
ping 192.168.10.10
ping 192.168.10.1


