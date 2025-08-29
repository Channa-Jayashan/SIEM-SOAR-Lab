# Week 2: Elastic Stack Setup and Windows/Kali Configuration

## Overview
Week 2 focused on configuring the Ubuntu Server VM with the Elastic Stack (Elasticsearch, Kibana, Filebeat) for SIEM functionality, setting up Sysmon on the Windows 10 VM for endpoint logging, and preparing the Kali VM for attack simulation.

## Detailed Steps

### 1. Installing and Configuring Elastic Stack on Ubuntu
- Accessed Ubuntu via SSH: `ssh specxy@192.168.1.101` (host CMD log in `logs/week1-2-ssh-logs.txt`).
- Updated system and installed Java:

sudo apt update && sudo apt upgrade -y
sudo apt install default-jdk -y

- Added Elastic repository and installed components:

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list
sudo apt update
sudo apt install elasticsearch kibana filebeat -y

- Edited Elasticsearch config (`/etc/elasticsearch/elasticsearch.yml`):

network.host: 0.0.0.0
discovery.type: single-node

- Started and enabled services:

sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch
sudo systemctl enable kibana && sudo systemctl start kibana
sudo systemctl enable filebeat && sudo systemctl start filebeat

- Set up passwords:

sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto

- Issue: `curl -X GET "http://localhost:9200"` returned "Empty reply from server".
- **Resolution**: Edited `elasticsearch.yml` to include config changes, restarted service (`sudo systemctl restart elasticsearch`), and retested successfully.
- Configured Filebeat (`/etc/filebeat/filebeat.yml`) to send system logs to Elasticsearch (default module enabled).

### 2. Setting Up Windows 10 VM with Sysmon
- Installed Windows 10 x64 VM: 2GB RAM, 32GB disk, attached ISO.
- Updated Windows, downloaded Sysmon.zip.
- Installed Sysmon via CMD (admin mode):

cd C:\Sysmon
sysmon64 -accepteula -i

- Verified logs in Event Viewer > Microsoft > Windows > Sysmon > Operational.
- Issue: Sysmon installation failed with "access denied".
- **Resolution**: Ran CMD as Administrator, reinstalled, and it worked.

### 3. Setting Up Kali VM
- Imported Kali .ova file (latest version) into VMware.
- Allocated 2GB RAM, connected to SIEM-LAN.
- Updated Kali:

- Verified logs in Event Viewer > Microsoft > Windows > Sysmon > Operational.
- Issue: Sysmon installation failed with "access denied".
- **Resolution**: Ran CMD as Administrator, reinstalled, and it worked.

### 3. Setting Up Kali VM
- Imported Kali .ova file (latest version) into VMware.
- Allocated 2GB RAM, connected to SIEM-LAN.
- Updated Kali:

sudo apt update && sudo apt upgrade -y

- Verified network connectivity to pfSense (192.168.1.1).

### 4. Testing and Integration
- Accessed Kibana at http://192.168.1.101:5601 from host (via pfSense port forwarding).
- Checked service statuses:

- Verified network connectivity to pfSense (192.168.1.1).

### 4. Testing and Integration
- Accessed Kibana at http://192.168.1.101:5601 from host (via pfSense port forwarding).
- Checked service statuses:

sudo systemctl status elasticsearch
sudo systemctl status kibana
sudo systemctl status filebeat

- All services were active (e.g., Elasticsearch since Aug 27, 2025).
- Issue: Log truncation in `elasticsearch-setup-passwords auto` output (cut off at ~205k chars).
- **Resolution**: Noted full output in `logs/week1-2-ssh-logs.txt`, summarized passwords set (redacted for security), and re-ran command to confirm.

## Errors and Resolutions
- **Error: Elasticsearch Curl Failure**
- **Description**: `curl` command returned empty reply.
- **Cause**: Missing or incorrect `network.host` setting.
- **Resolution**: Added `network.host: 0.0.0.0` and `discovery.type: single-node` to `elasticsearch.yml`, restarted service.
- **Error: Sysmon Access Denied**
- **Description**: Installation failed on Windows VM.
- **Cause**: Insufficient privileges.
- **Resolution**: Ran CMD as Administrator, reinstalled Sysmon.
- **Error: Log Truncation**
- **Description**: `elasticsearch-setup-passwords` output truncated.
- **Cause**: Command output exceeded log capture limit.
- **Resolution**: Saved full log to file, summarized key actions (password setup) in docs.

## Notes
- Ubuntu IP confirmed as 192.168.1.101 (ens33 interface).
- Mobile hotspot routing worked up to 192.168.25.2 (traceroute timeouts beyond due to mobile network).
- Screenshots saved: Kibana dashboard, Sysmon logs, Kali terminal.

[Back to README](../README.md)