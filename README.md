# Splunk-Lab

## Project overview

The goal of this project was to simulate a realistic enterprise environment and perform cyber attacks against Active Directory infrastructure, and detect malicious activity using Splunk. The primary focus was to **ingest** and **analyse logs**.

## Network Architecture

<img width="890" height="670" alt="image" src="https://github.com/user-attachments/assets/efa9acbe-fbdf-407a-960f-234ffdc9b826" />

## Network configuration 

- Network: 192.168.10.0/24
- Snort Server: 192.168.10.10
- Domain controller: 192.168.10.5
- Domain: cyber.local
- Windows 10: 192.168.10.100
- Kali Linux: 192.168.10.10

# Splunk installation on Ubuntu server
## Step 1 Download Splunk from official website 

```bash
wget -O splunk-10.2.1-c892b66d163d-linux-amd64.tgz "https://download.splunk.com/products/splunk/releases/10.2.1/linux/splunk-10.2.1-c892b66d163d-linux-amd64.tgz"
```
This command downloads the Splunk Enterprise 10.2.1 installation package from the official Splunk website.

**wget** = linux command to download file from internet.

**-O** = Used to select a name for the downloaded file.

## Step 2 Extract downloadedd file
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

After first startup user need to create user credentials (username and password) which is required login to splunk web interface.

## Start Splunk on system boot

```bash
sudo /opt/splunk/bin/splunk enable boot-start -user (username)
```

This command tells Splunk Enterprise to automatically start when the system boots and to run under a specific user account.



