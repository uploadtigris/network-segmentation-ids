# Initial Setup of Laptop 2 - Suricata Laptop

Ubuntu Server LTS has been installed via USB installer made with Balena Etcher

## Setting my user to run as sudo
```
sudo usermod -aG sudo <username>
```

## Setting a static IP for the latop
```
sudo nano /etc/netplan/00-installer-config.yaml
```
apply changes
```
sudo netplan apply
ip a show enp0s31f6
```

## keep the laptop running even with the lid closed
```
sudo nano /etc/systemd/logind.conf
```
uncommenting the following lines
```
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
restarting the service to apply
```
sudo systemctl restart systemd-logind
```

### Installing Suricata

We will install Suricata on bare-metal as Suricata needs low-level access to the NICs & Docker will add extra overhead without much benefit.

```
sudo add-apt-repository ppa:oisf/suricata-stable
sudo apt update
sudo apt install -y suricata
```

confirm installation
```
suricata --build-info | head -20
```

successful!

### Configuring in-line bridge mode

view network devices
```
ip link show
```

update `/etc/netplan/00-installer-config.yaml` with the following configuration`
```yaml
network:
  version: 2
  ethernets:
    enp0s31f6:
      dhcp4: false
      dhcp6: false
      match:
        macaddress: 8c:04:ba:0a:b6:7d
      set-name: enp0s31f6
    enxf4f951f28406:
      dhcp4: false
      dhcp6: false
  bridges:
    br0:
      interfaces: [enp0s31f6, enxf4f951f28406]
      addresses:
        - 192.168.1.189/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses: [192.168.1.1]
      parameters:
        stp: false
        forward-delay: 0
  wifis: {}
```

## point Suricata at br0

```
sudo nano /etc/suricata/suricata.yaml
```
modify the interface to be br0
```
af-packet:
  - interface: br0
    cluster-id: 99
    cluster-type: cluster_flow
    defrag: yes
```
Modify homenet to match LAN range
```
HOME_NET: "[192.168.1.0/24]"
```

running config in test mode
```
sudo suricata -T -c /etc/suricata/suricata.yaml -v

```
output:
```
...
Warning: detect: No rule files match the pattern /var/lib/suricata/rules/suricata.rules
```
solution: downloading rules with suricata-update
```
sudo suricata-update
```

### Testing live capture
```
sudo suricata -c /etc/suricata/suricata.yaml -i br0
```
on homelab computer, running
```
ping -c 10 8.8.8.8
```

### adding user to the suricata group
```
sudo usermod -aG suricata user
```

Suricata is now successfuly working as an inline bridge, capturing packets and ready to send them to Wazuh Manager. 
