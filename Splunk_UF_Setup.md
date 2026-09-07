# Splunk Universal Forwarder Deployment & Log Ingestion on Ubuntu

A comprehensive guide and lab documentation for installing, configuring, and deploying a **Splunk Universal Forwarder (UF)** on an Ubuntu Linux environment, connecting it to a **Splunk Enterprise Indexer**, and configuring automated log ingestion for system logs (`/var/log/syslog`).

---

## 📋 Architecture Overview

* **OS Environment:** Ubuntu Linux (CLI)
* **Splunk Universal Forwarder Directory:** `/opt/splunkforwarder`
* **Splunk Enterprise Indexer Port:** `9997` (Receiving Port)
* **UF Management Port:** `8090` (Configured to avoid collision with Splunk Enterprise default `8089`)
* **Target Index:** `custome_index`
* **Log Source Monitored:** `/var/log/syslog`

---

## 🚀 Deployment & Configuration Steps

### 1. Downloading & Extracting Universal Forwarder

Due to CLI download limitations, the package was fetched manually using `wget` directly on the server:

```bash
# Fetch Splunk Universal Forwarder package
wget -O splunkfowarder-10.4.3-4174a2deda5d-linux-amd64.tgz "[https://download.splunk.com/products/universalforwarder/releases/10.4.3/linux/splunkfowarder-10.4.3-4174a2deda5d-linux-amd64.tgz](https://download.splunk.com/products/universalforwarder/releases/10.4.3/linux/splunkfowarder-10.4.3-4174a2deda5d-linux-amd64.tgz)"

# Extract archive to /opt/
tar -xvzf splunkfowarder-10.4.3-4174a2deda5d-linux-amd64.tgz -C /opt/
2. Initial Setup & Management Port Configuration
When starting the Universal Forwarder for the first time, port 8089 was already bound by Splunk Enterprise. The management port was successfully shifted to 8090.

Bash
# Start Splunk Universal Forwarder and accept EULA
/opt/splunkforwarder/bin/splunk start --accept-license
Management Port Change: Selected 8090

Administrative Credentials: Configured admin user account for UF management.

3. File Ownership & Permission Alignment
To resolve file system ownership conflicts between root and the ubuntu system user:

Bash
# Assign ownership to ubuntu user
chown -R ubuntu:ubuntu /opt/splunkforwarder
4. Connecting UF to Splunk Indexer & Adding Log Monitor
Connected the Universal Forwarder to forward data to the local Splunk Indexer listening on port 9997, and enabled monitoring on /var/log/syslog mapped to custome_index.

Bash
# 1. Add Forwarder Receiver (Port 9997)
/opt/splunkforwarder/bin/splunk add forward-server 127.0.0.1:9997 -auth admin:'YOUR_PASSWORD'

# 2. Add System Log Monitor target
/opt/splunkforwarder/bin/splunk add monitor /var/log/syslog -index custome_index -auth admin:'YOUR_PASSWORD'
🔍 Verification & Ingestion Validation
Navigated to Splunk Web UI -> Search & Reporting App.

Switched the time picker to Last 24 hours (historical batch query to avoid real-time search overhead).

Executed the following Search Processing Language (SPL) query:

Splunk SPL
index="custome_index"
Results:
Events Ingested: Over 1,000+ system events indexed successfully.
```
<img width="757" height="431" alt="img" src="https://github.com/user-attachments/assets/82b54bb0-7c02-4105-8b14-0caaac91ae10" />


Metadata Verified:

host: ubuntu

source: /var/log/syslog

sourcetype: syslog

💡 Troubleshooting & Key Takeaways
Management Port Collisions: When running both Splunk Enterprise and Splunk Universal Forwarder on the same host, the management port (8089) must be reconfigured (e.g., to 8090) for the UF.

Special Characters in CLI Passwords: Passwords containing special bash characters (such as $) must be wrapped in single quotes ('p@$$w0rd') to prevent variable expansion errors in Linux terminals.

Real-time Search Timeouts: Running queries in Real-time mode on resource-constrained environments can lead to search dispatch timeouts; using fixed historical timeframes (e.g., Last 24 hours) ensures fast and reliable result retrieval.
