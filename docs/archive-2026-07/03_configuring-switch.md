# Configuring Managed Switch

In PFsense navigating to Status > DHCP Leases
- no leases appearing

Changed all VLAN to communicate over OPT1 (igc2)

Directly connected ethernet cable to managed switch
- set management laptop's IP to static address
- Connected to switch with default https://192.168.0.239

Switching > VLAN > added all VLANs from README.md
- all VLANs have Tagging all for the Router and Exclude all except for the Trusted VLAN
- The Trusted VLAN has Tagging all for the Router and Untag all for the WAP

