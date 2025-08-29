# Week 1: Lab Design and Initial VM Setup

## Overview
Week 1 focused on designing the network topology and setting up the initial virtual machines (VMs) for the SIEM/SOAR home lab using VMware Workstation. The goal was to establish a secure, isolated environment with pfSense as the firewall, Ubuntu Server as the SIEM base, Windows 10 for endpoint monitoring, and Kali for attack simulation.

## Detailed Steps

### 1. Setting Up VMware Workstation
- Installed VMware Workstation Player (free version) on the Windows 11 host laptop (16GB RAM, Intel processor).
- Verified hardware virtualization was enabled in BIOS (restarted laptop, entered BIOS via F2, enabled Intel VT-x).
- Created a new folder `C:\SIEM-Lab-VMs` to store VM files.

### 2. Designing Network Topology
- Planned a topology with pfSense as the gateway (WAN bridged to mobile hotspot, LAN as internal network).
- Used VMware's networking:
  - Created a Custom VMnet (named "SIEM-LAN") for the internal network (DHCP range 192.168.1.100-200 via pfSense).
  - Set pfSense WAN to Bridged Adapter (mapped to host WiFi for internet access).
- Other VMs (Ubuntu, Windows 10, Kali) connected to SIEM-LAN.

### 3. Creating and Installing pfSense VM
- Downloaded pfSense CE ISO (latest AMD64 version).
- New VM: 1GB RAM, 16GB disk, attached ISO.
- Installed pfSense with default settings, assigned WAN (Bridged) and LAN (SIEM-LAN) interfaces.
- Accessed web GUI at https://192.168.1.1 (admin/pfsense), configured DHCP on LAN (192.168.1.1), and allowed outbound WAN traffic.
- Issue: Initial boot looped due to incorrect network adapter selection.
  - **Resolution**: Changed adapter type to "e1000" in VMware settings, restarted VM, and it booted successfully.

### 4. Creating Ubuntu Server VM
- Downloaded Ubuntu 24.04.3 LTS ISO.
- New VM: 4GB RAM, 50GB disk, attached ISO.
- Installed Ubuntu Server, set hostname to "siemserver", created user "specxy", enabled OpenSSH.
- Connected to SIEM-LAN, received IP 192.168.1.101 (verified with `ip a`).
- Issue: SSH connection failed initially with "connection timed out".
  - **Resolution**: Ensured pfSense LAN rule allowed SSH (port 22), restarted Ubuntu VM, and reconnected successfully.

### 5. Initial Testing
- Pinged 192.168.1.1 (pfSense) from Ubuntu to confirm network.
- Attempted internet access (e.g., `ping 8.8.8.8`), which routed through mobile hotspot (traceroute showed 192.168.25.2).

## Errors and Resolutions
- **Error: pfSense Boot Loop**
  - **Description**: After installation, pfSense VM rebooted into a loop.
  - **Cause**: Incorrect network adapter (default VMXNET3 incompatible).
  - **Resolution**: Switched to e1000 adapter in VMware settings, reinstalled pfSense.
- **Error: SSH Connection Timeout**
  - **Description**: Couldn’t SSH to Ubuntu from host.
  - **Cause**: Missing firewall rule in pfSense.
  - **Resolution**: Added LAN rule to allow TCP 22, restarted Ubuntu network service.

## Notes
- Used mobile hotspot as the internet source, with pfSense handling routing.
- Screenshots saved: VMware interface, pfSense dashboard, Ubuntu terminal (`ip a`).

[Back to README](../README.md)