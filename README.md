# network-segmentation-ids

A segmented home network. management, trusted, IoT, and quarantine zones enforced with VLANs and firewall policy. Instrumented with Suricata for network intrusion detection. Suricata alerts feed the Wazuh SIEM for correlation, mirroring the VPC segmentation and flow-log analysis in the cloud projects. The write-up documents the segmentation policy, the threat each boundary mitigates, and proof that key detections fire against generated test traffic.
