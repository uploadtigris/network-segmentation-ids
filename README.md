# network-segmentation-ids

A repeatable image-hardening pipeline: Packer builds the base image, Ansible applies CIS benchmark controls, and OpenSCAP scores compliance before and after. The same Ansible role hardens EC2 in aws-secure-baseline, and Wazuh continuously verifies the result at runtime. Each control documents the before-risk, the change, and the residual risk, including the controls I deliberately skip and why.
