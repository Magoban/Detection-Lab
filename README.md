# SOC DETECTION HOME LAB

## Objective
[Brief Objective - Remove this afterwards]

This SOC home lab aims to simulate a real-world security monitoring and incident response environment. This setup allows for:

* Log collection, analysis, and correlation for security monitoring.

* Endpoint and network threat detection using EDR and IDS.

* Network segmentation and traffic filtering with a firewall.

* Hands-on experience with SIEM, EDR, IDS, and other security tools.

* Simulating cyberattacks and understanding their impact on an enterprise network.

* Incident response practice with a ticketing system.

### Skills Learned
[Bullet Points - Remove this afterwards]

* SIEM Administration (Graylog: log collection, analysis, alerting, and visualization)

* Endpoint Detection and Response (EDR) (OSSEC: monitoring endpoints for suspicious activity)

* Intrusion Detection System (IDS) (Suricata: detecting and analyzing malicious network traffic)

* Firewall Management (PfSense: network segmentation, firewall rules, VLANs)

* Log Forwarding & Parsing (Rsyslog, Sidecar, and OSSEC log forwarding)

* Threat Hunting & Security Analysis

* Incident Response & Management (Planned integration of TheHive ticketing system)

* Virtualization & Network Configuration (VMware, VirtualBox, Bridged and Internal Network Configurations)

* Cybersecurity Attack Simulations (Kali Linux: running attacks like port scanning, vulnerability scanning, and brute force)
### Tools Used

* Virtualization: VMware & VirtualBox (Used to deploy all components in a virtual lab setup)

* SIEM: Graylog (Installed on VMware)

* EDR: OSSEC (Installed on VirtualBox, connected to target machines)

* IDS: Suricata (Installed on VirtualBox, monitoring Target VLAN, logs forwarded to Graylog)

* Firewall: PfSense (Network segmentation and security policies enforced)

* Log Forwarding: Rsyslog, Sidecar (Ubuntu and Windows target machines)

* Security Analysis & Attacks: Kali Linux (Simulating real-world attacks for analysis)

* Incident Response: TheHive (Planned ticketing system for managing security incidents)

## Steps
1️⃣ SIEM - Graylog Setup

Installed Graylog on VMware to avoid MongoDB-VirtualBox conflicts.

Configured Sidecar and Rsyslog to forward logs from target machines.

Created inputs to receive logs from OSSEC, Suricata, and PfSense.

Configured alerts and dashboards for log visualization and monitoring.

*Ref 1: Network Diagram*
