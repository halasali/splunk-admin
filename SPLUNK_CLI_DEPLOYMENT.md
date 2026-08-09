## 🚀 Manual Deployment of Splunk Apps & Add-ons via CLI (Linux Server)
📌 Overview
This repository documents the production-grade installation and deployment workflow for Splunk Enterprise Apps and Add-ons (TAs) on a headless Ubuntu Linux Server.

In enterprise environments, GUI-based application management via Splunk Web is restricted due to strict security policies. This project demonstrates transferring Splunk app packages from a host Windows machine to a remote Ubuntu server via SFTP (WinSCP) and installing them manually via the Linux Command Line Interface (CLI).

🏗️ Technical Architecture & Environment
Splunk Server: Ubuntu Linux Server (CLI-only / Headless)

IP Address: 192.168.100.64

Management Host: Windows Desktop

Transfer Protocol: SFTP / SSH (Port 22 via OpenSSH Server)

Target Splunk App Directory: /opt/splunk/etc/apps/

🛠️ Step-by-Step Implementation Workflow
1. Enabling Remote SFTP Access
Installed and configured openssh-server on the Ubuntu host to enable secure file transfers:

Bash
sudo apt update
sudo apt install openssh-server -y
sudo ufw allow ssh
2. Package Transfer via WinSCP
Downloaded the required Splunk Apps (.tgz / .spl) from Splunkbase onto the Windows client and transferred them to the Ubuntu server user directory (/home/ubuntu/) over SFTP.

📸 Transfer Verification (WinSCP Screenshot)
Below is the file transfer verification showing the uploaded Splunk App packages in /home/ubuntu/:

<img width="401" height="331" alt="image2" src="https://github.com/user-attachments/assets/5d1fbfd5-eb77-4487-be2a-29862ba5aa7f" />

3. CLI Extraction & Installation
Extracted the transferred archives directly into Splunk's core applications directory:

Bash
# Extract Cisco Networks App & Add-on packages to Splunk apps directory
sudo tar -xvf ~/app-for-cisco-networks*.tgz -C /opt/splunk/etc/apps/
sudo tar -xvf ~/deprecated-add-on-for*.tgz -C /opt/splunk/etc/apps/
4. Permission Hardening (Crucial Step)
Adjusted directory ownership recursively to ensure the splunk system user retains full access permissions over the newly installed applications:

Bash
# Fix ownership and permissions
sudo chown -R splunk:splunk /opt/splunk/etc/apps/
sudo chmod -R 755 /opt/splunk/etc/apps/
5. Service Restart & Verification
Restarted the Splunk daemon (splunkd) to compile configurations and activate the newly added applications:

Bash
sudo /opt/splunk/bin/splunk restart
🎯 Key Takeaways & Best Practices
User Context & Permissions: Applications extracted using root or sudo will inherit elevated permissions, preventing Splunk from reading configurations unless ownership is reassigned to splunk:splunk.

Scalability: CLI-based deployment is essential for managing multi-node clusters (Search Head Clusters & Indexers) where Web UI app installation is disabled by design.
