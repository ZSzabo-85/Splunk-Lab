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

## Tools Used
* **SIEM:** Splunk Enterprise (Log collection and analysis)
* **Endpoint Telemetry:** Microsoft Sysmon (Advanced process and network monitoring)
* **Operating Systems:**
    * Ubuntu Server (Splunk Host)
    * Windows Server 2022 (Domain Controller)
    * Windows 10 (Client Endpoint)
    * Kali Linux (Attacker)
* **Attack Simulation:** Hydra (Brute-force) and Remmina (RDP Client)
* **Environment:** Active Directory (cyber.local)

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
  - [Remote Access Configuration and Verification](#remote-access-configuration-and-verification)
- [Attack Simulation: RDP Brute Force & RDP Connection](#attack-simulation-rdp-brute-force--rdp-connection)
- [Detection & SIEM Analysis](#detection--siem-analysis)
- [Key Takeaways](#key-takeaways)
- [Troubleshooting](#troubleshooting)

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

SPL
```
index=endpoint EventCode=4624 Logon_Type=10
```
# Attack Simulation: RDP Brute Force & RDP Connection
In this scenario, I simulated an external attack originating from a Kali Linux machine to gain unauthorized access to the Windows 10 client.

## Reconnaissance: 
Performed nmap scan to verify that the RDP port (TCP 3389) was open and accessible.

## Brute-Force: 
Created a dictionary file (password.txt) and utilized hydra to conduct a targeted password attack against awhite user account.

- **Command:** hydra -t 4 -V -f -l awhite -p password.txt rdp://192.168.10.100

## Access: 
Upon successful identification of the password, I utilized Remmina to establish a Remote Desktop session using the compromised credentials.

# Detection & SIEM Analysis
After successfully authenticating as awhite, I analyzed the Windows Event Logs in Splunk to identify the attacker.

- **Failed Logons:** Captured multiple instances of EventCode 4625 indicating the failed brute-force attempts from the Kali IP.
- 
SPL
```
index=endpoint EventCode=4625 
| stats count as failed_attempt by Account_Name, Source_Network_Address
| sort -failed_attempt
```
![Failed Logons](https://github.com/user-attachments/assets/62708946-bc9a-4942-9677-0331a1fff00b)

- **Analysis:** I aggregated failed logon events by account and source IP address to identify suspicious authentication activity. Many failures originatd from loopback addresses (127.0.0.1 / ::1) which represent local system activity. On the other hand the activity from the external IP (192.168.10.150) allowed me to identify the attacker's IP address responsible for brute-force attempts."

- **Successful Logon:** Identified the successful RDP logon event.

SPL
```
index="endpoint" EventCode=4624 (Logon_Type=2 OR Logon_Type=3 OR Logon_Type=7) Source_Network_Address=192.168.10.150
| table _time, Account_Name, Logon_Type, EventCode, Source_Network_Address
```
![Successful Logon with Logon_Types 2,3,7](https://github.com/user-attachments/assets/57700ac3-2ce7-4dac-ae2e-9cd52c6ad42a)

SPL
```
index="endpoint" EventCode=4624 Logon_Type IN (10)
| table _time, Account_Name, Logon_Type, EventCode, Source_Network_Address
```

![Successful Logon with Logon type 10](https://github.com/user-attachments/assets/5e126d09-ea7a-4430-b253-d54ed9cf284a)



- **Analysis:** This query was used to identify successful logon events.

  - **Logon Type 3 (Network):** Indicates successful network authentication.

  - **Logon Type 7 (Unlock):** The attacker unlocked a previously locked workstation session using valid credentials.

  - **Logon Type 10 (Remote Interactive):** Confirmed a full RDP session was established.
 
  ## Key Takeaways

  ## Troubleshooting

  
During this lab, several issues were encountered:

Domain join failure due to incorrect DNS configuration

Can you help to rewrite to more professional way

Here are a few more professional ways to phrase it, depending on the tone you want:

Simple and professional

During the lab, several issues were encountered, including a domain join failure caused by an incorrect DNS configuration.

Formal

Several issues were identified during the lab, most notably a domain join failure resulting from an incorrect DNS configuration.

Technical report style

During the lab exercise, multiple issues were encountered. One of the primary issues was a domain join failure, which was traced to an incorrect DNS configuration.

Concise

A domain join failure was encountered during the lab due to a DNS misconfiguration.

Root-cause focused

During the lab, a domain join failure occurred and was subsequently determined to be caused by an incorrect DNS configuration.


Example of data ingestion problem 

Splunk Data Ingestion: Data was not appearing in the endpoint index.
​Cause: I mistakenly named the configuration file input.conf instead of inputs.conf.
​Solution: Renamed the file to inputs.conf in the \local\ directory and restarted the Splunk Forwarder service. Verified that the index name matched exactly between the forwarder and the server.

You could document it in a professional incident-report format like this:

Data Ingestion Issue
Issue: Endpoint data was not being ingested into the designated Splunk index.

Cause: The Splunk Universal Forwarder configuration file was incorrectly named input.conf instead of inputs.conf, preventing the forwarder from processing the input configuration.

Resolution: Renamed the configuration file to inputs.conf in the local directory and restarted the Splunk Universal Forwarder service. Additionally, verified that the index name configured on the forwarder matched the destination index on the Splunk server. Data ingestion resumed successfully after the changes were applied.

Or, for a concise lab report:

Data Ingestion Failure: Endpoint data was not appearing in the target Splunk index.

Root Cause: Incorrect configuration filename (input.conf instead of inputs.conf).

Resolution: Renamed the file to inputs.conf, restarted the Splunk Universal Forwarder service, and confirmed index name consistency between the forwarder and Splunk server. Data ingestion was successfully restored.


Make one when win 10 cannot join to domain

Domain Join Failure
Issue: A Windows 10 workstation was unable to join the Active Directory domain.

Cause: The workstation was configured with an incorrect DNS server, preventing it from locating the domain controller and resolving Active Directory services.

Resolution: Updated the network adapter settings to use the correct DNS server associated with the Active Directory environment. Verified connectivity to the domain controller and successfully joined the workstation to the domain after refreshing the network configuration.

Outcome: The Windows 10 workstation was successfully joined to the domain, and domain authentication and Group Policy processing functioned as expected.

Concise Version:

Domain Join Failure: Windows 10 workstation could not join the Active Directory domain.

Root Cause: Incorrect DNS configuration prevented the workstation from locating the domain controller.

Resolution: Corrected the DNS settings, verified domain controller connectivity, and successfully completed the domain join process.


DNS Resolution: The Windows 10 client could not join the domain initially.
​Solution: Verified that the Windows 10 client's primary DNS was set to the Domain Controller's IP (192.168.10.5) to ensure proper resolution of the cyber.local domain
.

Here's a more professional version:

DNS Resolution Issue
Issue: The Windows 10 client was unable to join the Active Directory domain during the initial attempt.

Cause: The client was not configured to use the Domain Controller as its primary DNS server, preventing proper resolution of the cyber.local domain.

Resolution: Verified and updated the Windows 10 client's primary DNS setting to the Domain Controller's IP address (192.168.10.5). Once proper DNS resolution was established, the client was able to successfully locate the domain controller and join the cyber.local domain.

Outcome: Domain join completed successfully after correcting the DNS configuration.


Is thatbthe reason win 10 could not connect to domain? 

Yes. In an Active Directory environment, incorrect DNS configuration is one of the most common reasons a Windows 10 client cannot join a domain.

When a computer joins a domain, it must locate a domain controller by querying DNS for Active Directory service records (SRV records). If the client is using the wrong DNS server (for example, a public DNS server such as Google DNS or a router's DNS service), it will be unable to resolve the domain and find the domain controller.

Typical symptoms include errors such as:

"An Active Directory Domain Controller for the domain could not be contacted."

"The specified domain either does not exist or could not be contacted."

Failure to resolve the domain name (e.g., cyber.local).

For your lab report, you could write:

DNS Resolution Issue
Issue: The Windows 10 client was unable to join the cyber.local Active Directory domain.

Cause: The client was not configured to use the Domain Controller (192.168.10.5) as its primary DNS server, preventing it from resolving the cyber.local domain and locating the domain controller.

Resolution: Configured the Windows 10 client to use 192.168.10.5 as its primary DNS server and verified successful domain name resolution. The client was then able to locate the domain controller and join the domain successfully.

This explanation is technically accurate and aligns with how Active Directory domain joins work.


So first DNS Server was 8.8.8.8

Yes, if the Windows 10 client was initially configured to use 8.8.8.8 (Google Public DNS), that would explain why it could not join the cyber.local Active Directory domain.

Google's DNS servers can resolve public internet domains, but they do not host information about your internal Active Directory domain. As a result, the client would be unable to locate the domain controller through the DNS records required for Active Directory.

You could document it as follows:

- **DNS Resolution Issue**
 - **Issue:** The Windows 10 client was unable to join the cyber.local Active Directory domain.

 - **Cause:** The client was initially configured to use Google's public DNS server (8.8.8.8) instead of the Domain Controller's DNS. This prevented the client from joining to the domain.

 - **Solution:** Updated the client's primary DNS server to the Domain Controller's IP address (192.168.10.5). The client was then able to successfully join the domain.

