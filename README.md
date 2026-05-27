# Splunk-Lab

## Project overview

This project simulates a small enterprise Active Directory environment to generate and analyse logs using Splunk.

The lab was built to:
- Collect Windows Event Logs
- Forward endpoint telemetry to Splunk
- Configure Sysmon for enhanced logging
- Analyse authentication and remote login events
- Practice using SPL queries for log analysis

The primary focus of the project was log ingestion, endpoint monitoring, and event analysis using Splunk.

## Network Architecture

<img width="890" height="670" alt="image" src="https://github.com/user-attachments/assets/efa9acbe-fbdf-407a-960f-234ffdc9b826" />

## Network configuration 

| Device | IP Address |
|---|---|
| Splunk Server | 192.168.10.10 |
| Domain Controller | 192.168.10.5 |
| Windows 10 Client | 192.168.10.100 |
| Kali Linux Attacker | 192.168.10.150 |

- **Domain:** cyber.local  
- **Network:** 192.168.10.0/24

# Installing Splunk Enterprise on Ubuntu Server
## Step 1 - Download Splunk Enterprise

```bash
wget -O splunk-10.2.1-c892b66d163d-linux-amd64.tgz "https://download.splunk.com/products/splunk/releases/10.2.1/linux/splunk-10.2.1-c892b66d163d-linux-amd64.tgz"
```
This command downloads the Splunk Enterprise 10.2.1 installation package from the official Splunk website.

**wget** = linux command to download file from internet.

**-O** = Used to select a name for the downloaded file.

## Step 2 Extract downloaded file
```bash
tar -xvzf splunk-10.2.1-c892b66d163d-linux-amd64.tgz
```
**tar** = Linux command used to extract or create archive files (.tar / .tgz)

**-x** = extract files from an archive

**-v** = show files being extracted (verbose)

**-z** = use gzip compression/decompression

**-f** = specifies the filename of the archive


## Start Splunk and Accept licence agreement 
```bash
./splunk start --accept-license
```
This command starts Splunk and automatically accepts the license agreement.

After first startup user need to create user credentials (username and password) to access splunk web interface.

## Start Splunk on system boot

```bash
sudo /opt/splunk/bin/splunk enable boot-start -user (username)
```

This command tells Splunk Enterprise to automatically start when the system boots and to run under a specific user account.

# Install Splunk Universl Forwarder and Sysmon to Windows 10 and Windows Server 2022

## Download and install Splunk Universal Forwarder.

It forwards endpoints' logs to Splunk Server.

## Download and Configure Sysmon

For Sysmon configuration download and use configuration files available publicly (for his project Olaf Hartong configuration was used).

### Create inputs.conf file 











