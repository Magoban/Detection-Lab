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
## 2️⃣ Configuring Windows and Ubuntu Target to Send Logs to Graylog using Graylog Sidecar
### Step 1: Install and Configure Graylog Sidecar on Windows
Graylog Sidecar is a lightweight agent that helps manage log collectors like **Winlogbeat or NxLog** to send logs to Graylog.
### Download & Install Sidecar
1. Download Graylog Sidecar from [Graylog Sidecar GitHub Release](https://github.com/Graylog2/collector-sidecar/releases)
2. Run the installer and install the program.
3. After installation, open PowerShell (Admin) and navigate to the install directory:
```powershell
cd "C:\Program Files\Graylog\Sidecar"
```
4. Install Sidecar as a service:
```powershell
.\graylog-sidecar.exe -service install
```
5. Start the Sidecar service:
```powershell
.\graylog-sidecar.exe -service start
```
### Configure Sidecar to Connect to Graylog
1. Open the Sidecar configuration file (ensure you are still in the Sidecar directory):
```powershell
notepad sidecar.yml
```
2. Modify these lines:
```yaml
server_url: "http://<GRAYLOG_IP>:9000/api/"
server_api_token: "<YOUR_GRAYLOG_API_TOKEN>"
node_id: "windows-client"
```
* Replace <Graylog_IP> with the IP address of your Graylog server.
* Replace <YOUR_GRAYLOG_API_TOKEN> with an API token from Graylog. Follow the steps below to get an API token from Graylog:
 * In the Graylog web UI, select **system**
 * Select **Sidecars**
 * CLick **Create Token**
3. Save the file and exit.
4. Restart the Sidecar service:
```powershell
Restart-Service graylog-sidecar
```
### Configure Sidecar in Graylog Web UI
1. Log into Graylog Web UI (http://<GRAYLOG_IP>:9000).
2. Go to: **System > Sidecars**.
3. You should see your Windows machine listed. ✅
4. Click "**Configuration**" next to the Windows client.
5. Select "**Winlogbeat**" as the Collector Backend.
6. Click "**Assign**".
### Configure Winlogbeat in Sidecar
1. Go to System > Sidecars > Configuration.
2. Click Create New Configuration.
3. Set the following settings:
 * Name: *Windows Logs*
 * Collector Backend: *Winlogbeat*
 * **Configuration**: Edit the **output.logstash:
   hosts: ["${user.graylog_host}:5044"]** to **output.logstash:
   hosts: ["<GRAYLOG_IP>:5044"]**
You can also remove logs that you don't want to be collected to reduce the number of logs to be monitored and collected.
4. Click **save**
### Step 4: Start Sending Logs
1. Go back to **System > Sidecars**.
2. Click "**Restart**" next to your Windows client.
3. After a few minutes, logs should appear in **Graylog > Search**
### Verify Logs in Graylog
* Go to **Graylog Web UI > Search**.
* Use this query:
```bash
source:windows-client-name
```
You should start seeing logs from your **Windows Target Machine**
### Step 2: Install and Configure Graylog Sidecar on Ubuntu
### Download & Install Sidecar
1. Download Graylog Sidecar from [Graylog Sidecar GitHub Release](https://github.com/Graylog2/collector-sidecar/releases)
2. Install Sidecar using dpkg
```bash
cd ~/Downloads
sudo dpkg -i graylog-sidecar_1.5.0-1_amd64.deb #or whichever version you downloaded or whatever the file name is.
```
3.  Fix missing dependencies (if needed):
```bash
sudo apt update && sudo apt install -f
```
4. Edit the Sidecar configuration file:
```bash
sudo nano /etc/graylog/sidecar/sidecar.yml
```
Find and update these lines:
```yaml
server_url: "http://<GRAYLOG_IP>:9000/api/"
server_api_token: "<YOUR_GRAYLOG_API_TOKEN>"
node_id: "ubuntu-client"
```
* Input your Graylog server IP address.
* Follow the same steps as Windows to get a Graylog API Token for the Ubuntu.
* Save and exit (CTRL+X, then Y, then ENTER).
### Start the Sidecar Service
```bash
sudo systemctl enable graylog-sidecar
sudo systemctl start graylog-sidecar
sudo systemctl status graylo-sidecar
```
Now, you might run into an error like this :
**Failed to enable unit: Unit file graylog-sidecar.service does not exist.
Failed to start graylog-sidecar.service: Unit graylog-sidecar.service not found.**
This error can be because Graylog Sidecar service is missing or not installed correctly.
To fix this error, do the following:
#### Verify Installation: Make sure your installation is successful
```bash
dpkg -l | grep graylog-sidecar
```
* If it shows an output, the sidecar is installed successfully
* If there's no output, the sidecar isn't installed properly and you will need to install it again.
#### Check If the Service Exists
```bash
ls -l /etc/systemd/system/graylog-sidecar.service
```
* If the service exist, try reloading systemd
```bash
sudo systemctl daemon-reload
```
Then, start the sidecar again
* If the file does NOT exist, it means the Sidecar service file is missing. Follow these steps to manually create the service
Create the service:
```bash
sudo nano /etc/systemd/system/graylog-sidecar.service
```
Paste the following inside the file:
```ini
[Unit]
Description=Graylog Sidecar
After=network.target

[Service]
ExecStart=/usr/bin/graylog-sidecar
Restart=always
User=root
Group=root

[Install]
WantedBy=multi-user.target
```
Save and exit (CTRL+X, then Y, then ENTER).
Reload systemd and enable Sidecar:
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now graylog-sidecar
sudo systemctl status graylog-sidecar
```
It should now be running and active
### Configure Graylog to Receive Logs from Ubuntu
#### Register Ubuntu in Graylog
1. Go to Graylog Web UI (http://<GRAYLOG_IP>:9000).
2. Navigate to **System > Sidecars**.
3. You should see "**ubuntu-client**" in the list. ✅
4. Click "**Configuration**" next to the Ubuntu client.
5. Select **Filebeat** as the Collector Backend.
6. Click "**Assign**".
### Configure Filebeat in Sidecar
#### Create a Filebeat Configuration in Graylog
1. Go to **System > Sidecars > Configuration**.
2. Click **Create New Configuration**.
3. Set:
 * Name: Ubuntu Logs
 * Collector Backend: Filebeat
4. Edit configuration
Find the filebeat.inputs section and add the following inside it:
```yaml
 - type: log
    enabled: true
    paths:
      - /var/log/syslog
      - /var/log/auth.log
      - /var/log/kern.log
```
5. Save the changes and restart the Sidecar service on the Ubuntu Target Machine:
```bash
sudo systemctl restart graylog-sidecar
```
### Verify Logs in Graylog
1. Go to Graylog Web UI > Search.
2. Use this query:
```ini
source:ubuntu-client-name
```
Logs should be appearing and you have successfully integrated your target machines to send logs to Graylog
## 2️⃣ Setting Up OSSEC (EDR) on VirtualBox
Download and install Ubuntu-22.04.5-server on VirtualBox
[Ubuntu 22.04.5 server version](https://releases.ubuntu.com/22.04/ubuntu-22.04.5-live-server-amd64.iso)
# Step 1: Download and install OSSEC server
## 📌  Installation Steps:
For ease of use, ssh into the machine on a desktop Operating System (Windows or Linux). This will allow you to copy and paste commands easily.
 ### Update the Ubuntu system
```sh
sudo apt-get update && sudo apt-get upgrade -y
```
### Install Required Dependencies
```sh
sudo apt install -y curl unzip make gcc policycoreutils python3 python3-pip inotify-tools
```
### Additional Dependencies
```sh
sudo apt update && sudo apt install -y \
    curl unzip make gcc policycoreutils python3 python3-pip \
    inotify-tools libpcre2-dev libssl-dev libevent-dev \
    build-essential zlib1g-dev libz-dev libsystemd-dev \
    libcap-ng-dev
```
### Download OSSEC Source Code
```sh
curl -O https://github.com/ossec/ossec-hids/archive/refs/tags/3.7.0.tar.gz
```
### Extract the Downloaded File
```sh
tar -xvzf 3.7.0.tar.gz
cd ossec-hids-3.7.0
```
### Start the OSSEC Installer
```sh
Start the OSSEC Installer
```
* When prompted, select: server mode.
* Enter the IP of your Ubuntu server machine as the OSSEC server IP.
* Accept default options unless you have a specific reason to change them.
### Start the OSSEC Service
```sh
sudo /var/ossec/bin/ossec-control start
```
### Check the Status of the OSSEC Service
```sh
sudo /var/ossec/bin/ossec-control status
```
**Note:** A problem I ran into was the OSSEC remote-d not running. OSSEC remote-d allows OSSEC agents to communicate with OSSEC server. Here are some fixes for the problem:
### Manually start the remote-d
```sh
sudo /var/ossec/bin/ossec-remoted -d
```
Then check the status of the OSSEC server again
### Stop and start the OSSEC server again
```sh
sudo /var/ossec/bin/ossec-control stop
sudo /var/ossec/bin/ossec-control start
```
If none of these fixes work for you, go ahead to install and register the agents. Sometimes, OSSEC remote-d fails because there are no agents registered on its list of allowed IPs.
# Step 2: Install OSSEC Agents on Target Machines
### 🔹 Install OSSEC Agent on Ubuntu Target Machine
If your Ubuntu Desktop (target machine) is a fresh installation, update the system packages and install the required dependencies before beginning the installation of the OSSEC Agent.
### Download & Install OSSEC Agent
```sh
curl -O https://github.com/ossec/ossec-hids-agent/archive/refs/tags/3.7.0.tar.gz
tar -xvzf 3.7.0.tar.gz
cd ossec-hids-agent-3.7.0
sudo ./install.sh
```
* Choose "agent" mode when prompted.
* Enter the OSSEC Server IP when prompted.
* Accept default options unless you have a specific reason to change them.
### Start and Verify the Status of the OSSEC Agent
```sh
sudo /var/ossec/bin/ossec-control start
sudo /var/ossec/bin/ossec-control status
```
### 🔹 Install OSSEC Agent on Windows Target Machine
Download the OSSEC agent from:
🔗 [OSSEC Official Download Page](https://www.ossec.net/download-ossec/)
* Run the installer and choose "Agent" mode.
* Enter the OSSEC Server's IP address when prompted.
* Complete the installation.
* Start the OSSEC Agent
* Open Windows Services (services.msc).
* Find OSSEC HIDS Agent.
* Right-click → Start.
# Step 3: Register and Connect Agents to OSSEC Server
📌 These steps should be done on the OSSEC Server machine.

### Add the Ubuntu Agent to the OSSEC Server
```sh
sudo /var/ossec/bin/manage_agents
```
* Choose: A (Add Agent)
* Enter:
  Agent Name → ubuntu-target
  Agent IP → Ubuntu Target IP
  Confirm
Get the Agent Key from the output.
### Add the Windows Agent to the OSSEC Server
Repeat the above steps for Windows, using its IP and hostname.
### Import Agent Key on Ubuntu Target
On the Ubuntu Target Machine, Run:
```sh
sudo /var/ossec/bin/agent-auth -m <OSSEC_SERVER_IP> -p 1515
```
**Note:** replace <OSSEC_SERVER_IP) with the IP address of the Ubuntu machine the OSSEC server is installed on.
### Import Agent Key on Windows Target
* Open OSSEC Agent Manager.
* Go to Manage Keys → Import Key (Copy-Paste from OSSEC Server).
* Save and restart the OSSEC agent.
# Step 4: Configure OSSEC to Send Logs to Graylog
On the OSSEC sever,
### Edit OSSEC Configuration File
```sh
sudo nano /var/ossec/etc/ossec.conf
```
Add this section inside *<global>* :
```sh
<remote>
    <connection>syslog</connection>
    <port>514</port>
    <protocol>udp</protocol>
    <allowed-ips>GRAYLOG_IP</allowed-ips>
</remote>
```
Replace GRAYLOG_IP with the actual IP of your Graylog server.

Save and exit (CTRL+X, then Y, then ENTER).
### Restart the OSSEC Server to Apply Changes
```sh
sudo /var/ossec/bin/ossec-control restart
```
# Step 5: Configure Graylog to Receive OSSEC Logs
Log into the Graylog Web UI for this step
### Create a New Syslog UDP Input in Graylog
1. Log into Graylog Web GUI
2. Go to: System → Inputs
3. Click "Launch new Input"
4. Select "Syslog UDP"
5. Configure:
 * Bind Address: 0.0.0.0
 * Port: 514
 * Allow overriding date: ✅ Yes
6. Click Save.
# Step 6: Verify Logs Are Reaching Graylog
### Monitor Logs on OSSEC Server
```sh
sudo tail -f /var/ossec/logs/ossec.log
```
You should see log-forwarding messages
### Check Logs in Graylog
Go to Graylog Web GUI → Search
Use this query:
```bash
source:OSSEC_SERVER_IP
```
If logs appear, OSSEC is successfully integrated with Graylog!🎉


