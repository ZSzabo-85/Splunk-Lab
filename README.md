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

## Table of Contents
- [Network Architecture](#network-architecture)
- [Network Configuration](#network-configuration)
- [Installing Splunk Enterprise on Ubuntu Server](#installing-splunk-enterprise-on-ubuntu-server)
  - [Step 1: Download Splunk Enterprise](#step-1---download-splunk-enterprise)
  - [Step 2: Extract Downloaded File](#step-2-extract-downloaded-file)
  - [Start Splunk and Accept License](#start-splunk-and-accept-licence-agreement)
  - [Start Splunk on System Boot](#start-splunk-on-system-boot)
- [Install Splunk Universal Forwarder and Sysmon](#install-splunk-universal-forwarder-and-sysmon-on-windows-10-and-windows-server-2022)
  - [Configure inputs.conf](#configure-inputsconf-for-splunk-universal-forwarder)
  - [Restart Splunk Forwarder Service](#restart-splunk-forwarder-service)
- [Configure Splunk Server](#configure-splunk-server)
  - [Create Index](#create-index)
  - [Enable Data Receiving](#enable-data-receiving)
- [Verify Data Ingestion](#verify-data-ingestion)
- [Active Directory](#active-directory)
  - [Windows 10 Domain Integration](#windows-10-domain-integration)

## Network Architecture

![Network Architecture Diagram](https://github.com/user-attachments/assets/efa9acbe-fbdf-407a-960f-234ffdc9b826)

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

Go to **Search & Reporting** and run `index=endpoint` to confirm that Splunk is receiving data from your endpoints. The results below confirm connectivity from both the Windows 10 client and the Windows Server:

```spl
index=endpoint
```
![Splunk Host Ingestion Verification](https://github.com/user-attachments/assets/07cc7b62-4e31-4096-85e0-19bfeac236c1)

# Active Directory

I prompted Windows Server 2022 to a Domain Controller for the cyber.local forest. This environment serves as the foundation for testing identity-based attacks and monitoring. 

- **Organizational Structure:** I created multiple Organizational Units (OUs)—IT, HR, and Finance—to simulate a production environment.
- **Identity Management:** I created user accounts to perform attack. The awhite user account was specifically configured to serve as the primary target for cyber attack simulations.

## Windows 10 Domain Integration

To integrate the client machine into the domain, I configured the WIN10-DESKTOP to communicate with the Domain Controller (192.168.10.5).
- **DNS Configuration:** I updated the network settings on the Windows 10 client, setting the primary DNS server to the Domain Controller's IP address to resolve the cyber.local domain.
- **Domain Join:** After resolving the DNS issue, I joined the machine to the domain via the System Properties menu.
-  **Verification:** Following a system restart, I performed a successful login using the domain credentials for awhite user account. The user profile folder was automatically generated for the account upon first login, confirming the successful connection with Active Directory.

## Remote Access Configuration and Verification

To perform remote attack simulations (like RDP brute-force), I configured the WIN10-DESKTOP to accept remote connections.
- **Enable RDP:** Enabled "Allow remote connections to this PC" via System Properties.
- **Access Control:** Explicitly configured Remote Desktop user permissions to allow the domain-joined awhite account access. This allowed me to test my security detections against realistic remote attack scenarios.

To validate the configuration, I performed a remote login to the WIN10-DESKTOP using awhite domain account credentials.
- **Connectivity Check:** Initiated a session via the Remote Desktop Protocol (RDP) using the mstsc client.
- **Verification:** Successfully established the remote session and confirmed connectivity by verifying the machine's hostname and IP address via the command line (ipconfig).
- **Session Termination:** Properly terminated the session, confirming that domain-authenticated remote access is functional and ready for security testing.
- Detection validation: Validated the remote logon event in Splunk to ensure the activity was captured:

Splunk SPL
```
index=endpoint EventCode=4624 Logon_Type=10
```

