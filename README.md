# SIEM-SOAR-Lab

A home lab implementation of SIEM/SOAR for log collection and incident response using Elastic Stack, Wazuh Integration, TheHive SOAR Setup, and simulated attacks/analyses.

## Project Overview
- **Objective**: Build a SIEM/SOAR lab to collect, analyze, and respond to logs, showcasing defensive cybersecurity skills.
- **Duration**: Augest 2025 – September 2025 (8 weeks).
- **Tools/Environment**: VMware Workstation (host on Windows 11 laptop), pfSense firewall, Ubuntu 24.04 LTS server (Elastic Stack), Windows 10 endpoint (Sysmon), Kali Linux attacker. Internet via mobile hotspot (routing through 192.168.25.x subnet).
- **Repo Structure**:
  - `docs/`: Detailed weekly write-ups.
  - `screenshots/`: Visuals of setups and interfaces.
  - `configs/`: Configuration file snippets.
  - `logs/`: Command and SSH logs.
- **Progress**: Week 1-2 completed (Lab Design and VM Setup). Upcoming: Weeks 3-4 (Wazuh Integration).

## Week 1-2: Lab Design and VM Setup
### Setup Plan
| Aspect | Instruments | Implementation |
|--------|----------------------------|-----------------------|
| Type 2 Hypervisor | VMware Workstation | VMware Workstation (as shown in screenshots/vmware-vms-overview.jpg) – Chosen for better performance on host hardware (16GB RAM Intel processor). |
| Network Topology | pfSense (WAN bridged to WiFi, LAN internal), Ubuntu/Windows/Kali on internal net. | Same, but using VMware's Bridged (for WAN to mobile hotspot) and VMnet (custom internal for LAN). pfSense gateway: 192.168.1.1. Ubuntu IP: 192.168.1.101 (ens33 interface). Internet routing: Traceroute shows mobile hotspot at 192.168.25.2. |
| VMs | pfSense, Ubuntu Server, Windows 10, Kali. | Created in VMware: Kali (powered on), pfSense (powered on), Ubuntu Server 64-bit (powered on, hostname: siemserver, user: specxy), Windows 10 x64 (powered off during log capture). Allocated RAM: ~4GB Ubuntu, 2GB others. |

### Detailed Steps and Commands
Accessed Ubuntu VM via SSH from host: `ssh specxy@192.168.1.101` (note: Log shows initial connect to .102 – is a prior DHCP lease; it manually updated to .101).

1. **VM Creation and Networking in VMware**:
   - Installed VMware Workstation (free Player version sufficient for lab).
   - Created VMs: File > New Virtual Machine > Used ISOs for pfSense, Ubuntu 24.04.3 LTS, Windows 10; imported Kali .ova.
   - Network: pfSense with 2 adapters (Bridged to WiFi for WAN, Custom VMnet for LAN). Other VMs on Custom VMnet (DHCP via pfSense: 192.168.1.100-200 range).
   - Test: Ping 8.8.8.8 from VMs (routes through pfSense > mobile hotspot).
![VMware VMs Overview](screenshots/vmware-vms-overview.jpg)

2. **pfSense Installation and Firewall Config**:
   - Installed pfSense CE (latest AMD64 ISO).
   - Web GUI: https://192.168.1.1 (admin/pfsense).
   - Rules: WAN outbound allow all (with logging); LAN allow HTTP/HTTPS to Elastic ports (9200, 5601).
![pfSense Dashboard](screenshots/pfSense-dashboard.jpg)

3. **Ubuntu Setup with Elastic Stack**:
   - Installed Ubuntu Server, enabled OpenSSH.
   - Updated system and installed Java:
![Elasticsearch Status](screenshots/Elasticsearch-status.jpg)
![Kibana and Filebeat Status](screenshots/Kibana_Filebeat-status.jpg)
![Kibana Dashboard](screenshots/Kibana_Dashboard.jpg)

sudo apt update && sudo apt upgrade -y
sudo apt install default-jdk

- Added Elastic repo and installed components:

wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list<br>
sudo apt update<br>
sudo apt install elasticsearch kibana filebeat -y

- Edited `/etc/elasticsearch/elasticsearch.yml` (via `sudo nano`):

network.host: 0.0.0.0 <br>
discovery.type: single-node

- Started services:

sudo systemctl enable elasticsearch && sudo systemctl start elasticsearch <br>
sudo systemctl enable kibana && sudo systemctl start kibana <br>
sudo systemctl enable filebeat && sudo systemctl start filebeat

- Set passwords (auto mode; log truncated – redacted sensitive output):

sudo /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto

- Verification: `curl -X GET "http://localhost:9200"` (initially failed due to config; resolved after edits). Status checks show services running (e.g., Elasticsearch active since Aug 27, 2025).
- Filebeat config: Edited `/etc/filebeat/filebeat.yml` for system modules and Elasticsearch output.

![Ubuntu IP Config](screenshots/UbuntuServer-terminal-ip_a.jpg)  
![Elasticsearch Status](screenshots/Elasticsearch-status.jpg)

4. **Windows 10 with Sysmon**:
- Installed Windows 10 x64.
- Downloaded Sysmon.zip, installed via CMD: `sysmon64 -accepteula -i`.
- Logs in Event Viewer > Microsoft > Windows > Sysmon.

5. **Kali Setup**:
- Imported .ova, updated: `sudo apt update && sudo apt upgrade -y`.

### Troubleshooting and Notes
- SSH fingerprint warning resolved with 'yes'.
- Curl to Elasticsearch initially empty (fixed by config edits and restart).
- Internet connectivity: VMs route through pfSense to mobile hotspot; traceroute timeouts beyond hotspot normal for mobile networks.
- Full SSH log: See [logs/week1-2-ssh-logs.txt](logs/week1-2-ssh-CommandLogs.txt) for raw commands (truncated sections summarized here).
- Resources Used: Elastic.co docs, VMware tutorials (instead of VirtualBox).

## Week 3-4: Wazuh Integration

## Phase 1 Summary: Wazuh Server Install
- Upgraded Elastic to 8.19.3.
- Added Wazuh repo and installed components.
- Configured OpenSearch and Dashboard for SSL/local access.
- Integrated Filebeat for Wazuh-Elastic forwarding.
- Verified with curl queries showing alerts in wazuh-alerts indices.
- Kibana visualization confirmed working.

### Phase 1: Install Wazuh Server on Ubuntu VM (Completed)
Integrated Wazuh with the existing Elastic Stack for centralized alerting and log forwarding. Kibana dashboard now visualizes Wazuh logs successfully.

#### Steps Taken (Based on Logs from 2025-08-30)
1. **System Update and Upgrade**:
   - SSH to Ubuntu: `ssh specxy@192.168.1.101`.
   - Ran `sudo apt update && sudo apt upgrade` to update Elastic components to 8.19.3 (elasticsearch, kibana, filebeat) and other packages.

2. **Add Wazuh Repository**:
   - Added GPG key: `curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo apt-key add -`.
   - Added repo: `echo "deb https://packages.wazuh.com/4.x/apt/ stable main" | sudo tee -a /etc/apt/sources.list.d/wazuh.list`.
   - Updated packages: `sudo apt update`.

3. **Install Wazuh Components**:
   - Installed Wazuh Indexer: `sudo apt-get install wazuh-indexer`.
   - Configured `/etc/wazuh-indexer/opensearch.yml` for single-node setup (see configs/opensearch.yml).
   - Started service: `sudo systemctl daemon-reload && sudo systemctl enable wazuh-indexer && sudo systemctl start wazuh-indexer`.
   - Installed Wazuh Manager: `sudo apt-get install wazuh-manager`.
   - Started service: `sudo systemctl daemon-reload && sudo systemctl enable wazuh-manager && sudo systemctl start wazuh-manager`.
   - Generated certificates: Used `wazuh-certs-tool.sh` to create SSL certs for secure comms.
   - Installed Filebeat with Wazuh module: `sudo apt-get install filebeat=8.19.3`, configured template, and enabled module.
   - Installed Wazuh Dashboard: `sudo apt-get install wazuh-dashboard`.
   - Configured `/etc/wazuh-dashboard/opensearch_dashboards.yml` (see configs/opensearch_dashboards.yml).
   - Started service: `sudo systemctl daemon-reload && sudo systemctl enable wazuh-dashboard && sudo systemctl start wazuh-dashboard`.

4. **Integrate with Elastic Stack**:
   - Configured Filebeat to forward Wazuh alerts to Elasticsearch (hosts: localhost:9200, with elastic credentials).
   - Verified integration: Indices like `wazuh-alerts-*` created; sample alert query shows logs (e.g., sudo execution alert).
   - Kibana now visualizes Wazuh logs (e.g., via Discover or Dashboard tabs).

5. **Verification**:
   - Service statuses: All active (wazuh-manager, wazuh-indexer, filebeat, wazuh-dashboard).
   - API check: `curl -X GET "https://localhost:55000" -u admin:admin -k` (returns Wazuh status).
   - Indices: `curl -u specxy:sp3cxy -k "https://192.168.1.101:9200/_cat/indices?v"` lists wazuh-alerts and filebeat indices.
   - Alert query: `curl -u specxy:sp3cxy -k "https://192.168.1.101:9200/wazuh-alerts-*/_search?size=1&pretty"` confirms log ingestion.

![Wazuh Manager Status](screenshots/wazuh-manager-status.jpg)  
![Wazuh Indexer Status](screenshots/wazuh-indexer-status.jpg)  
![Wazuh Alerts](screenshots/wazuh-alerts-query.jpg)

#### Issues and Resolutions
- **Wazuh Indexer Start Failure**: Missing certs during initial setup.  
  - *Resolution*: Generated certificates using provided scripts and reconfigured `opensearch.yml`.  

- **Dashboard Service Crash**: Config pointed to Wazuh Indexer instead of Elasticsearch.  
  - *Resolution*: Adjusted `opensearch_dashboards.yml` to match Elasticsearch backend.  

- **Disk Space Limitation**: Ubuntu VM partition was full.  
  - *Resolution*: Expanded LVM partition from 19G → 38G using `lvextend` and `resize2fs`.  

- **Unstable SSH Sessions**: Mobile hotspot occasionally dropped connections.  
  - *Resolution*: Reconnected manually; no service-level impact.  

Full install logs: [logs/week3-4-phase1-install-logs.txt](logs/week3-4-phase1-install-logs.txt).  
Resources Used: Wazuh.com documentation (all-in-one deployment guide).

### Upcoming in Week 3-4
- Phase 2: Install Wazuh Agents on Windows and Kali.
- ...

## License
MIT License (see LICENSE file).

