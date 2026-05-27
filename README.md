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

## Step 2 Extract downloaded file
```bash
tar -xvzf splunk-10.2.1-c892b66d163d-linux-amd64.tgz
```
## Start Splunk and Accept licence agreement 
```bash
./splunk start --accept-license
```
This command starts Splunk and automatically accepts the license agreement.

On the first startup, user is prompted to create credentials (username and password) to access th Splunk web interface.

## Start Splunk on system boot

```bash
sudo /opt/splunk/bin/splunk enable boot-start -user analyst
```
This command tells Splunk Enterprise to automatically start when the system boots and run under the specified user (analyst) account.

# Install Splunk Universal Forwarder and Sysmon on Windows 10 and Windows Server 2022

## Download and Install Splunk Universal Forwarder

The Splunk Universal Forwarder was installed on both Windows 10 and Windows Server 2022 endpoints. It is used to forward logs from the endpoints to the Splunk server for centralised analysis.

## Download and Configure Sysmon

Sysmon was installed to generate detailed endpoint telemetry.

A publicly available configuration file was used (Olaf Hartong’s Sysmon configuration) to improve logging visibility.

Download the raw configuration file and save it as `sysmonconfig.xml`.

Open PowerShell as Administrator and navigate to the Sysmon directory, then run:

```powershell
.\Sysmon64.exe -i ..\sysmonconfig.xml
```

## Configure inputs.conf for Splunk Universal Forwarder

Open Notepad as Administrator and create an `inputs.conf` file.

The default location for configuration files is:
`C:\Program Files\SplunkUniversalForwarder\etc\system\default`

However, it is **best practice not to modify** files in the `default` directory.

Instead, the `inputs.conf` file should be created in the `local` directory to ensure custom configurations are preserved during updates:

`C:\Program Files\SplunkUniversalForwarder\etc\system\local`

### inputs.conf (Splunk Universal Forwarder Configuration)

The following configuration defines which Windows Event Logs are forwarded to Splunk.

```ini
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```
## Configure Splunk Forwarder and Connect to Splunk Server

The `inputs.conf` file was saved to the following directory:

`C:\Program Files\SplunkUniversalForwarder\etc\system\local`

After modifying `inputs.conf`, the Splunk Universal Forwarder service must be restarted for changes to take effect.

### Restart Splunk Forwarder Service

Open **Services** as Administrator and locate **SplunkForwarder**.

- Ensure the service is running under **Local System** account
- If not, change the Log On settings accordingly
- Right-click the service and select **Restart**

This ensures the updated configuration is applied correctly.

---

## Configure Splunk Server

On the Windows 10 machine, open a web browser and navigate to:

`http://[Splunk_Server_IP]:8000`

Log in using the credentials created during Splunk installation on the Ubuntu server.

### Create Index

Navigate to:

**Settings → Indexes → New Index**

Create an index named:

`endpoint`

This must match the index defined in `inputs.conf`.

---

### Enable Data Receiving

Navigate to:

**Settings → Forwarding and receiving → Configure receiving → New Receiving Port**

- Set the port to `9997`
- Save the configuration

This port is used for receiving data from forwarders.

---

## Verify Data Ingestion

Go to **Search & Reporting** and run:

```spl
index=endpoint
```










