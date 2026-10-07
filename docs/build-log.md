# Segmentation Lab: Build Log

> [!CAUTION]
> The subnets and device IP addresses on this page are for demonstration only. They
> are not the ones I actually use. I keep the real addressing out of this repo for
> security reasons.

**Date:** 2026-10-07 to 2026-10-__  
**OS:** pfSense, Netgear GS308EP firmware, TP-Link EAP610 firmware, Raspberry Pi OS  
**Environment:** Homelab  
**Category:** Networking, VLANs, Firewall, DHCP, DNS  
**Status:** In progress

The design is in the [README](../README.md). This page is the working log: what I
did, in order, and what broke along the way. Screenshots live in [`../images/`](../images/).

---

## Situation

_What did the network look like before, and why change it?_

- Network before the build (addressing, which devices shared a segment):
- What prompted the build:
- First attempt, July 2026 (what was built, where it stalled):

## Task

Segment the home network into five VLANs with default-deny rules between them,
and prove each rule works from the restricted side.

| VLAN | Name | Subnet | Gateway | Who lives here |
|---|---|---|---|---|
| 1 | Mgmt | 10.0.1.0/24 | 10.0.1.1 | Switch (10.0.1.2), AP (10.0.1.3), recovery port |
| 20 | Trusted | 10.0.20.0/24 | 10.0.20.1 | Laptop, phone, PCs |
| 30 | IoT | 10.0.30.0/24 | 10.0.30.1 | Smart devices, hydroponics project |
| 40 | Guest | 10.0.40.0/24 | 10.0.40.1 | Visitors |
| 50 | Servers | 10.0.50.0/24 | 10.0.50.1 | Pi-hole (10.0.50.10), Latitude (10.0.50.20) |

Done when: every row of the test matrix below passes, or the failure is documented.

## Action

Tick each item when it is done and the evidence is saved. Add the date to each step.

### 0. Before touching anything — Date: ____

- [ ] Pick a time nobody else needs the internet
- [ ] Download the pfSense config backup (Diagnostics > Backup & Restore)
- [ ] Save the GS308EP switch config from its web UI
- [ ] Save the EAP610 config (System > Backup)
- [ ] Write down the current IPs: pfSense ____ · switch ____ · AP ____ · Pi-hole ____
- [ ] Laptop wired into the switch for the whole build

### 1. pfSense: VLANs and DHCP — Date: ____

- [ ] Create VLANs 20, 30, 40, 50 on the LAN NIC (Interfaces > Assignments > VLANs)
- [ ] Add each VLAN as an interface, enable it, name it TRUSTED / IOT / GUEST / SERVERS
- [ ] Give each interface a static IPv4 of 10.0.X.1/24
- [ ] Re-address LAN to 10.0.1.1/24 for Mgmt
- [ ] DHCP server on each interface, range .100 to .199
- [ ] DHCP DNS server: 10.0.50.10 on Trusted, IoT and Servers; 10.0.40.1 on Guest
- [ ] Static DHCP mapping: Pi-hole 10.0.50.10
- [ ] Static DHCP mapping: Latitude 10.0.50.20
- [ ] Evidence: screenshot of the VLAN list and each DHCP page

```
# notes, commands, output
```

**Screenshots**

_Placeholder: pfSense VLAN list_ — Interfaces > Assignments > VLANs, showing tags 20, 30, 40 and 50 on the LAN NIC  
<!-- ![pfSense VLAN list](../images/01_pfsense_vlans.png) -->

_Placeholder: pfSense interface assignments_ — TRUSTED, IOT, GUEST and SERVERS enabled with their gateway addresses  
<!-- ![pfSense interface assignments](../images/01_pfsense_interfaces.png) -->

_Placeholder: DHCP scope: Trusted_ — range, DNS server and gateway for VLAN 20  
<!-- ![DHCP scope: Trusted](../images/01_pfsense_dhcp_trusted.png) -->

_Placeholder: DHCP scope: IoT_ — range, DNS server and gateway for VLAN 30  
<!-- ![DHCP scope: IoT](../images/01_pfsense_dhcp_iot.png) -->

_Placeholder: DHCP scope: Guest_ — range and DNS server (pfSense resolver, not Pi-hole) for VLAN 40  
<!-- ![DHCP scope: Guest](../images/01_pfsense_dhcp_guest.png) -->

_Placeholder: DHCP scope: Servers_ — range plus the static mappings for the Pi-hole and the Latitude  
<!-- ![DHCP scope: Servers](../images/01_pfsense_dhcp_servers.png) -->

### 2. Switch: 802.1Q VLANs (GS308EP) — Date: ____

| Port | Device | Untagged (PVID) | Tagged |
|---|---|---|---|
| 1 | pfSense LAN (trunk) | 1 | 20, 30, 40, 50 |
| 2 | EAP610 (PoE) | 1 | 20, 30, 40 |
| 3 | Pi-hole | 50 | none |
| 4 | Latitude 7490 | 50 | none |
| 5 | Gaming PC | 20 | none |
| 6 | AI PC | 20 | none |
| 7 | Spare (Trusted) | 20 | none |
| 8 | Recovery port (Mgmt) | 1 | none |

- [ ] Enable Advanced 802.1Q VLAN (VLAN > 802.1Q > Advanced)
- [ ] Create VLANs 20, 30, 40, 50
- [ ] Set tagged and untagged membership per the port map
- [ ] Set PVID 50 on ports 3 and 4
- [ ] Set PVID 20 on ports 5, 6 and 7
- [ ] Remove ports 3 to 7 from VLAN 1
- [ ] Confirm ports 1, 2 and 8 are still untagged in VLAN 1
- [ ] Give the switch the static IP 10.0.1.2
- [ ] Confirm the switch UI is reachable from port 8 (the way back in)
- [ ] Evidence: screenshot of VLAN membership and the PVID table

```
# notes, commands, output
```

**Screenshots**

_Placeholder: Switch VLAN membership_ — tagged (T) and untagged (U) ports for each VLAN, matching the port map  
<!-- ![Switch VLAN membership](../images/02_switch_vlan_membership.png) -->

_Placeholder: Switch PVID table_ — PVID 50 on ports 3 and 4, PVID 20 on ports 5 to 7, PVID 1 on ports 1, 2 and 8  
<!-- ![Switch PVID table](../images/02_switch_pvid.png) -->

### 3. Access point: one SSID per VLAN (EAP610) — Date: ____

| SSID | VLAN | Band | Security |
|---|---|---|---|
| Home | 20 | 2.4 + 5 GHz | WPA2/WPA3 |
| IoT | 30 | 2.4 GHz only | WPA2-Personal |
| Guest | 40 | 2.4 + 5 GHz | WPA2, client isolation on |

- [ ] Give the AP the static IP 10.0.1.3, management VLAN untagged
- [ ] Map the home SSID to VLAN 20
- [ ] Create the IoT SSID on 2.4 GHz only, mapped to VLAN 30
- [ ] Map the Guest SSID to VLAN 40 with client isolation on
- [ ] Confirm a client on each SSID gets an address in the right subnet
- [ ] Evidence: screenshot of the SSID and VLAN settings

```
# notes, commands, output
```

**Screenshots**

_Placeholder: AP SSID to VLAN mapping_ — each SSID with its VLAN ID, band and security mode  
<!-- ![AP SSID to VLAN mapping](../images/03_ap_ssid_vlan.png) -->

_Placeholder: Wi-Fi clients on the right subnets_ — one client per SSID showing an address from its VLAN  
<!-- ![Wi-Fi clients on the right subnets](../images/03_client_addresses.png) -->

### 4. Pi-hole onto the Servers VLAN — Date: ____

- [ ] Move the Pi to switch port 3
- [ ] Confirm it comes up on 10.0.50.10 (change the static IP on the Pi itself if it is set there)
- [ ] Set Pi-hole to "Permit all origins" (Settings > DNS > Interface settings)
- [ ] Confirm a Trusted client resolves through the Pi-hole
- [ ] Evidence: Pi-hole dashboard at 10.0.50.10, query log showing clients from other VLANs

```
# notes, commands, output
```

**Screenshots**

_Placeholder: Pi-hole dashboard on the Servers VLAN_ — admin page loaded at the Pi-hole's Servers address  
<!-- ![Pi-hole dashboard on the Servers VLAN](../images/04_pihole_dashboard.png) -->

_Placeholder: Pi-hole interface setting_ — Settings > DNS with "Permit all origins" selected  
<!-- ![Pi-hole interface setting](../images/04_pihole_permit_all.png) -->

_Placeholder: Pi-hole query log_ — queries arriving from clients on other VLANs  
<!-- ![Pi-hole query log](../images/04_pihole_query_log.png) -->

### 5. Firewall rules — Date: ____

Rules are evaluated top-down per interface; first match wins.

- [ ] Create the alias RFC1918 = 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16
- [ ] TRUSTED: allow any
- [ ] IOT rule 1: allow UDP/TCP 53 to 10.0.50.10
- [ ] IOT rule 2: block to RFC1918
- [ ] IOT rule 3: allow to any
- [ ] GUEST rule 1: allow UDP/TCP 53 to 10.0.40.1
- [ ] GUEST rule 2: block to RFC1918
- [ ] GUEST rule 3: allow to any
- [ ] SERVERS rule 1: block to RFC1918
- [ ] SERVERS rule 2: allow to any
- [ ] LAN (Mgmt): leave the default allow
- [ ] Evidence: screenshot of the rule list on every interface
- [ ] Stretch: NAT port-forward on IOT redirecting outbound port 53 to the Pi-hole

```
# notes, commands, output
```

**Screenshots**

_Placeholder: RFC1918 alias_ — the three private ranges in one alias  
<!-- ![RFC1918 alias](../images/05_alias_rfc1918.png) -->

_Placeholder: Firewall rules: TRUSTED_ — allow any  
<!-- ![Firewall rules: TRUSTED](../images/05_rules_trusted.png) -->

_Placeholder: Firewall rules: IOT_ — DNS to Pi-hole, block RFC1918, allow any, in that order  
<!-- ![Firewall rules: IOT](../images/05_rules_iot.png) -->

_Placeholder: Firewall rules: GUEST_ — DNS to the gateway, block RFC1918, allow any, in that order  
<!-- ![Firewall rules: GUEST](../images/05_rules_guest.png) -->

_Placeholder: Firewall rules: SERVERS_ — block RFC1918, allow any  
<!-- ![Firewall rules: SERVERS](../images/05_rules_servers.png) -->

_Placeholder: NAT DNS redirect (stretch)_ — port-forward on IOT sending outbound port 53 to the Pi-hole  
<!-- ![NAT DNS redirect (stretch)](../images/05_nat_dns_redirect.png) -->

### 6. Move the hydroponics gear to IoT — Date: ____

- [ ] Join the Pico W to the IoT SSID
- [ ] Join the Pi Zero W to the IoT SSID
- [ ] Join the Gardyn to the IoT SSID
- [ ] Evidence: DHCP leases showing the devices on 10.0.30.x

**Screenshots**

_Placeholder: IoT DHCP leases_ — the Pico W, Pi Zero W and Gardyn holding addresses on the IoT VLAN  
<!-- ![IoT DHCP leases](../images/06_iot_dhcp_leases.png) -->

### 7. Test matrix — Date: ____

Screenshot every result.

| # | Test | From | Command | Expected | Actual | Pass |
|---|---|---|---|---|---|:-:|
| 1 | Got the right address | Each VLAN | `ipconfig` / `ip a` | 10.0.X.1xx | | ☐ |
| 2 | DNS through Pi-hole | Trusted, IoT | `nslookup doubleclick.net` | Blocked (0.0.0.0) | | ☐ |
| 3 | Guest can't see home | Guest phone | `ping 10.0.20.x` | Fails | | ☐ |
| 4 | IoT can't reach the server | IoT device | `curl http://10.0.50.20` | Fails | | ☐ |
| 5 | Trusted reaches the server | Laptop | browser to 10.0.50.20 | Loads | | ☐ |
| 6 | Mgmt locked down | Guest or IoT | browser to 10.0.1.2 | Fails | | ☐ |
| 7 | Internet everywhere | All VLANs | `ping 1.1.1.1` | Works | | ☐ |

**Screenshots**

_Placeholder: Test 1: right address_ — `ipconfig` / `ip a` output from a client on each VLAN  
<!-- ![Test 1: right address](../images/07_test_1_address.png) -->

_Placeholder: Test 2: DNS through Pi-hole_ — `nslookup doubleclick.net` returning 0.0.0.0 from Trusted and IoT  
<!-- ![Test 2: DNS through Pi-hole](../images/07_test_2_dns_blocked.png) -->

_Placeholder: Test 3: Guest can't see home_ — ping from a Guest phone to a Trusted address failing  
<!-- ![Test 3: Guest can't see home](../images/07_test_3_guest_isolated.png) -->

_Placeholder: Test 4: IoT can't reach the server_ — `curl` from an IoT device to the Latitude failing  
<!-- ![Test 4: IoT can't reach the server](../images/07_test_4_iot_blocked.png) -->

_Placeholder: Test 5: Trusted reaches the server_ — the server page loading from the laptop  
<!-- ![Test 5: Trusted reaches the server](../images/07_test_5_trusted_allowed.png) -->

_Placeholder: Test 6: Mgmt locked down_ — switch UI unreachable from Guest or IoT  
<!-- ![Test 6: Mgmt locked down](../images/07_test_6_mgmt_locked.png) -->

_Placeholder: Test 7: internet everywhere_ — `ping 1.1.1.1` succeeding from every VLAN  
<!-- ![Test 7: internet everywhere](../images/07_test_7_internet.png) -->

### 8. Write-up — Date: ____

- [ ] Network diagram finalized
- [ ] `network-segmentation-ids` README updated: zone table, port map, rule table, diagram, screenshots
- [ ] "Problems I hit" section written from the entries below
- [ ] Status changed to Completed (only after the test matrix passes)
- [ ] Portfolio site segmentation card updated

**Screenshots**

_Placeholder: Final network diagram_ — five VLANs, trunks, gateways, Pi-hole and the Latitude  
<!-- ![Final network diagram](../images/08_network_diagram.png) -->

## Result

_Fill in once the test matrix has been run._

- Outcome:
- Test matrix: __ of 7 passed
- What I would do differently:
- Lesson learned:

---

## Problems I hit

One STAR entry per problem. Copy the blank entry for each new one.

### Problem 1: No DHCP leases on the tagged interfaces (July 2026 attempt)

**Date:** 2026-07-__

**Situation** — _What were the symptoms?_

**Task** — _What needed to work?_

**Action** — _What did you check, in order, and what did you change?_

```
# commands and output
```

**Result** — _Root cause, fix, and what you would watch for next time._

### Problem 2: [title]

**Date:** 2026-10-__

**Situation** —

**Task** —

**Action** —

```
# commands and output
```

**Result** —

### Problem 3: [title]

**Date:** 2026-10-__

**Situation** —

**Task** —

**Action** —

```
# commands and output
```

**Result** —

---

**Tags:** `networking` `vlan` `802.1q` `pfsense` `dhcp` `dns` `pihole` `firewall` `homelab`
