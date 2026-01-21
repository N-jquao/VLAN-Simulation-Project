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
21. To set up the web server database, open the vm's network settings and select "LAN Segment"
22. Choose the "Internal_Lab" option from the dropdown and save
