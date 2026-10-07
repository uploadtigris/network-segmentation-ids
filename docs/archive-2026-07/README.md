# Archive: first attempt, July 2026

These notes describe my first try at segmenting the network. They are kept for the
record and do **not** describe the current design.

- They use a `192.168.1.x` scheme and a different VLAN layout (Mgmt 10, Trusted 20,
  IoT 30, Quarantine 40, Guest 50).
- The build stalled when no DHCP leases appeared on the tagged interfaces.
- The Suricata notes cover an inline-bridge experiment that is not part of the
  current build. Detection is phase 2 and has not been built.

The current design is in the [README](../../README.md) and the current work is in
the [build log](../build-log.md).

| File | What it covers |
|---|---|
| [01_configuring-suricata.md](01_configuring-suricata.md) | Ubuntu laptop set up as an inline Suricata bridge |
| [02_configuring-pfsense.md](02_configuring-pfsense.md) | pfSense install, first VLANs and DHCP |
| [03_configuring-switch.md](03_configuring-switch.md) | Managed switch VLANs, where the DHCP leases stopped appearing |
