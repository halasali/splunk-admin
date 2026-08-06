# splunk-admin
# Splunk Enterprise Architecture & Linux Server Hardening

This repository documents the installation, performance optimization, system hardening, and service configuration for a production-ready **Splunk Enterprise** deployment on Ubuntu Linux.

---

## 🏗️ Splunk Enterprise Architecture & Components

Splunk Enterprise consists of core operational components and specialized administration tools that handle the end-to-end data pipeline from ingestion to visualization.

### 1. Log Ingestion Tier (Forwarders)
* **Universal Forwarder (UF):** A lightweight agent installed on endpoints. It only collects and forwards raw logs without parsing, maintaining minimal CPU and RAM usage.
* **Heavy Forwarder (HF):** A full Splunk instance capable of parsing, filtering, transforming, and routing data before sending it to Indexers.

### 2. Processing & Storage Tier (Indexer)
* **Indexer:** The core storage engine and search processor.
  * **Parsing:** Breaks raw data into events, extracts timestamps, and creates line breaks.
  * **Indexing:** Writes compressed events to disk inside time-based storage structures called **Buckets**.
  * **Searching:** Executes search queries requested by the Search Head across local buckets.

### 3. Visualization & Management Tier (Search Head)
* **Search Head (SH):** Provides the web user interface (Web UI).
  * Receives SPL (Search Processing Language) queries from users.
  * Distributes queries across Indexers.
  * Consolidates and formats search results into Dashboards, Reports, and Alerts.

### 4. Administration Tier
* **Deployment Server (DS):** Centralized management server used to push configuration updates, apps, and `inputs.conf` files to Forwarders automatically.
* **License Manager (LM):** Tracks daily data ingestion volumes across Indexers against the Splunk license quota ($\text{GB/day}$).
* **Cluster Manager (formerly Cluster Master):** Manages Indexer clustering, data replication, and bucket copy availability.

---

## ⚡ System Hardening & Performance Optimization

Before deploying Splunk Enterprise on Linux, several kernel and system-level configurations must be applied to meet production standards and prevent bottlenecks.
!Screenshot 2026-08-06 122544.png
### 1. Disabling Transparent Huge Pages (THP)
* **What was changed?**  
  Disabled **Transparent Huge Pages (THP)** at the kernel level (`[never]`).
* **Why is this necessary?**  
  By default, Linux attempts to optimize memory by grouping standard memory pages ($4\text{ KB}$) into huge blocks ($2\text{ MB}$). However, Splunk performs aggressive, high-frequency, continuous read/write operations for indexing and searching. THP introduces memory allocation latency and CPU throttling, which can degrade Splunk's performance by up to $50\%$ or lead to system hangs. Disabling THP ensures fast, allocation-on-demand memory access.

### 2. Increasing Resource Limits (`ulimits`)
* **What was changed?**  
  Increased system resource constraints for file descriptors (`nofile`) and user processes (`nproc`) to higher values (or `unlimited`).
* **Why is this necessary?**  
  Splunk simultaneously manages thousands of open network connections, socket bindings, index files, and search processes. Standard Linux defaults (e.g., $1024$ open files) are insufficient and will cause `Too many open files` errors and indexing failures.
!image.png
---!image.png

## 🔒 Service Configuration & Boot-Start Setup

To ensure service continuity and enforce cybersecurity best practices:

```bash
sudo /opt/splunk/bin/splunk enable boot-start -user splunk
