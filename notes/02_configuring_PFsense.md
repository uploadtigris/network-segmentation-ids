# Installing and Configuring PFsense

Downloading USB installer from [PFsense website](https://www.pfsense.org/download/)
- version: AMD64 Memstick USB (Intel x64 Netgate appliances)
- bootable media made using Balena etcher

Switching Gfiber router to Bridge mode

Using RJ45 Ethernet cables: 
- Connected Gfiber router (bridge) to MiniPC (PFsense) to unmanaged switch to laptop (for web-GUI access)

Navigated to 192.168.1.1 for access into PFsense web GUI
- configured DNS to Cloudflare (1.1.1.1) - primary & Google (8.8.8.8) - secondary
- configured LAN inteface (because we do not want to leave is at 192.168.1.1 as this is insecure)

Checked for updates & rebooted the PFsense router

### VLANs

As described in the README.md of this repo, I am going to configure 4 VLANS:
- Management - 10
- Trusted - 20
- IoT - 30
- Quarantine - 40
- Guest - 50

Navigated to Interface > Assignments > VLANs
- configured the 5 VLANs

Under Interfaces > Interface Assignments
- added all VLANs

Right now these VLANs are only at the Link Layer, we are now going to bring them up to the Network Layer
- Interface Assignments > click interface name (ex. OPT1)

Configured DHCP ranges for each VLAN under: Services > DHCP Server > TRUSTED , for all VLANs

### Firewall Rules

Firewall > Rules

For each VLAN: Add > Action = Pass > Protocol = Any


