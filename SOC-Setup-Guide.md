# 🚀 SOC Home Lab Setup Guide

## 📌 Overview
* This SOC home lab is designed to simulate real-world security monitoring using:
* SIEM (Graylog) for log collection & correlation
* EDR (OSSEC) for endpoint monitoring
* IDS (Suricata) for intrusion detection
* Firewall (pfSense) for network segmentation
* Kali Linux for cyberattack simulations
* Incident Response (TheHive - Planned)

## 🛠️ Tools & Technologies
| Component  |	Tool Used	  | Purpose                                   |
|------------|--------------|-------------------------------------------|
| SIEM	     | Graylog	    | Log collection, correlation, and analysis |
| EDR	       | OSSEC	      | Endpoint monitoring for Windows & Linux   |
| IDS	       | Suricata	    | Detects network-based threats             |
| Firewall	 | pfSense	    | Network segmentation & security policies  |
| Target VMs |	Windows 10 & Ubuntu	| Simulating real-world workstations |
| Attack VM	 | Kali Linux	  | Simulating cyberattacks                    |
| Analyst VM | Ubuntu |     | Simulating real-word analyst workstation    |
| Incident Response | TheHive (Planned)	| Tracking & responding to security incidents |

## 📍 Lab Architecture & Network Segmentation on pfSense
Interface     | Network                  | Purpose  
-------------|--------------------------|------------------------------------  
Adapter 1    | Bridged Mode             | Allows Graylog to route traffic  
Adapter 2    | Security Management VLAN | Connected to OSSEC & Analyst Machine  
Adapter 3    | Target VLAN              | Connected to Suricata, Windows & Ubuntu targets  
Adapter 4    | Attack VLAN              | Connected to Kali Linux  

## 📌 Step-by-Step Installation & Configuration
1️⃣ **Installing Graylog (SIEM) on VMware**

🔹 Why VMware?

Graylog requires **MongoDB**, which has CPU conflicts with VirtualBox.

Download and install Ubuntu-22.04.5-desktop on VMware
[Ubuntu 22.04.5 desktop version](https://releases.ubuntu.com/22.04/ubuntu-22.04.5-desktop-amd64.iso)

📌 Installation Steps
```sh
# Update the Ubuntu system
sudo apt-get update && sudo apt-get upgrade -y
```
