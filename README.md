# SOC DETECTION HOME LAB

## Network Setup ##
![Mini-SOC Lab (4)](https://github.com/user-attachments/assets/a23d7033-5dad-46e2-84d0-c0e2b495ee44) *Ref 1: Network Diagram*


## Objective

This SOC home lab aims to simulate a real-world security monitoring and incident response environment. This setup allows for:

* Log collection, analysis, and correlation for security monitoring.

* Endpoint and network threat detection using EDR and IDS.

* Network segmentation and traffic filtering with a firewall.

* Hands-on experience with SIEM, EDR, IDS, and other security tools.

* Simulating cyberattacks and understanding their impact on an enterprise network.

* Incident response practice with a ticketing system.

### Skills Learned

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
![Screenshot (64)](https://github.com/user-attachments/assets/6fef7b4a-fce8-4730-bc65-64ecccf79ea1) *Ref 2: Virtualization setup*

* SIEM: Graylog (Installed on VMware)

* EDR: OSSEC (Installed on VirtualBox, connected to target machines)

* IDS: Suricata (Installed on VirtualBox, monitoring Target VLAN, logs forwarded to Graylog)

* Firewall: PfSense (Network segmentation and security policies enforced)

* Log Forwarding: Rsyslog, Sidecar (Ubuntu and Windows target machines)

* Security Analysis & Attacks: Kali Linux (Simulating real-world attacks for analysis)

* Incident Response: TheHive (Planned ticketing system for managing security incidents)

## Steps
1️⃣ SIEM - Graylog Setup

* Installed Graylog on VMware to avoid MongoDB-VirtualBox conflicts.

* Configured Sidecar and Rsyslog to forward logs from target machines.

* Created inputs to receive logs from OSSEC, Suricata, and PfSense.

* Configured alerts and dashboards for log visualization and monitoring.
![Screenshot (65)](https://github.com/user-attachments/assets/eb603765-d353-4ce2-8ec0-1ab34c6bea42) *Ref 3: Graylog Interface*


2️⃣ EDR - OSSEC Installation & Integration

* Installed OSSEC on VirtualBox and connected it to Security Management VLAN.

* Installed OSSEC agents on target machines (Ubuntu & Windows) and configured them to send logs to the OSSEC server.

* Forwarded OSSEC logs to Graylog for centralized monitoring.

3️⃣ IDS - Suricata Deployment & Configuration

* Installed Suricata on VirtualBox and attached it to the Target VLAN.

* Configured Suricata to monitor network traffic on the correct interface.

* Enabled rule sets for detecting malicious activities.

* Forwarded Suricata alerts to Graylog for analysis.

4️⃣ Firewall - PfSense Network Segmentation

* Installed PfSense on VirtualBox.

* Configured network segmentation using VLANs:

  * Adapter 1 (Bridged Mode): Connected to Graylog for routing ease.

  * Adapter 2 (Security Management VLAN): Connected to OSSEC and Analyst machine.

  * Adapter 3 (Target VLAN): Connected to Ubuntu & Windows target machines.

  * Adapter 4 (Attack VLAN): Connected to Kali Linux for attack simulations.

* Implemented firewall rules to allow/deny traffic between VLANs.

5️⃣ Target Machines & Log Forwarding

* Installed Ubuntu Desktop & Windows 10 on VirtualBox, attaching them to the Target VLAN.

* Configured Sidecar on Windows and Ubuntu to send logs to Graylog.

* Configured rsyslog for additional log forwarding.

6️⃣ Analyst Machine Setup

* Installed Ubuntu Desktop on VirtualBox and attached it to Security Management VLAN.

* Ensured access to Graylog and OSSEC with strict security controls.

* Prepared the machine for security analysis and incident response.

7️⃣ Attack Machine - Kali Linux for Cyber Attacks

* Installed Kali Linux on VirtualBox and connected it to the Attack VLAN.

* Conducted various attacks, including:

* Port scanning (Nmap)

* Vulnerability scanning (OpenVAS, John)

* Brute-force attacks (Hydra, Medusa)

* Observed attack logs in Graylog and OSSEC.

8️⃣ Incident Response - TheHive (Planned Setup)

* Plan to install TheHive for incident tracking and management.

* Integrate with Graylog to create alerts and track security events.

## Conclusion

This SOC home lab provides a hands-on approach to security monitoring, log analysis, and incident response. With the current setup, I have gained practical knowledge of setting up and managing a SOC environment, configuring SIEM, EDR, and IDS tools, and performing cybersecurity attack simulations.

Future enhancements will include:

* **Full deployment of TheHive for incident response tracking**

* **Refining alerting and detection rules for improved threat intelligence**

* **Exploring additional threat-hunting techniques using SIEM dashboards**

* **Automating response mechanisms for detected threats**

This portfolio is a documentation of my cybersecurity journey and a reference for others looking to build their own SOC home lab.
