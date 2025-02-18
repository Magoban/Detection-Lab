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

## 📌  Installation Steps:
 ### Update the Ubuntu system
```sh
sudo apt-get update && sudo apt-get upgrade -y
```
### Install dependencies
```sh
sudo apt-get install gnupg curl
```
### Import mongodb key
```sh
curl -fsSL https://www.mongodb.org/static/pgp/server-6.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-6.0.gpg \
   --dearmor
```
### Create the repository
```sh
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-6.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
```
### Update the package database
```sh
sudo apt-get update
```
### Install MongoDB
```sh
sudo apt-get install -y mongodb-org
```
### Enable, start and check the status of the service
```sh
sudo systemctl daemon-reload
sudo systemctl enable mongod.service
sudo systemctl start mongod.service
sudo systemctl status mongod.service
```
MongoDB should be running and active
### To avoid auto update of the MongoDB, use the command
```sh
sudo apt-mark hold mongodb-org
```
### Install Graylog DataNode. 
Data Node is a Graylog management tool for OpenSearch. This means you don't need to install OpenSearch separately for Graylog to manage logs properly.
```sh
wget https://packages.graylog2.org/repo/packages/graylog-6.1-repository_latest.deb
sudo dpkg -i graylog-6.1-repository_latest.deb
sudo apt-get update
sudo apt-get install graylog-datanode
```
### As displayed, the Linux setting vm.max_map_count needs to be set
```sh
sudo sysctl -w vm.max_map_count=262144
echo "vm.max_map_count = 262144" | sudo tee /etc/sysctl.d/99-graylog-datanode.conf
```
### Set The Password Secret
```sh
sudo sed -i "/^password_secret =/s/$/ $(< /dev/urandom tr -dc A-Z-a-z-0-9 | head -c${1:-96})/" /etc/graylog/datanode/datanode.conf
```
### Confirm the password_secret is set (You can open a new terminal Window for this. Leave this new Window opened or you can just copy the password_secret to your notepad)
```sh
grep password_secret /etc/graylog/datanode/datanode.conf
```
If you opened a new terminal Window for the previous step, return to the terminal window you've been using since the beginning. 
### Now let's enable and start the Graylog DataNode Service
```sh
sudo systemctl enable graylog-datanode.service
sudo systemctl start graylog-datanode
sudo systemctl status graylog-datanode.service
```
Data Node should be running and active
### Now, let's install the Graylog server
```sh
sudo apt-get install graylog-server
```
### Set admin password (The admin password you will set here is the password you will use to log into your Graylog web UI)
```sh
echo -n "Enter Password: " && head -1 </dev/stdin | tr -d '\n' | sha256sum | cut -d" " -f1
```
Copy the hash that is generated
### Modify the Graylog configurations file
```sh
sudo nano /etc/graylog/server/server.conf
```
Look for the root_password_sha2 = paste the admin password hash you copied here.
Go to your notepad, or if you didn't close the other terminal Window that has the password_secret, copy it. Then, on the server.conf file, look for the section password_secret = paste the password_secret here.
You can configure the time zone Graylog will use to your preference. To do this in the conf file, look for the #root_timezone = UTC. By default it is set to UTC, uncomment this by removing the #, then set the time zone to what you want. There is a link provided in the conf file that contains all the supported time zone formats. e.g., root_timezone = Africa/Lagos to set the time to Nigeria's time.
Next, change the http_bind_address to your Ubuntu machine IP address ( use "ip a" in a terminal window to check your machine's IP address ). Leave the port as:9000
Press ctrl+x then y and press Enter to save and exit the config file.
### Enable and start Graylog server
```sh
sudo systemctl daemon-reload
sudo systemctl enable graylog-server.service
sudo systemctl start graylog-server.service
sudo systemctl status graylog-server.service
```
### Setup additional information for the data node
```sh
tail -f /var/log/graylog-server/server.log
```
A URL will be provided as part of the output, click on it and begin the data node configuration as follows:
1. Configure the CA: Set the organization's name to "graylog_lab" without the quotation mark. Click to create CA.
2. Set the renewal policy as you see fit: I set it to be automatic and the certificate lifetime as 30 days.
3. Click to create policy and on the next page click to begin provisioning.
4. Once the provisioning is done, click to resume startup.
Now, your Graylog and data node should be running successfully. Log in with the username "admin" and the admin password you created during the installation steps.





