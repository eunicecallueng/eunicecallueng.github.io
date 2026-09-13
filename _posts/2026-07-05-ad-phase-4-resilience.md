---
title: "Phase 4: Architecture Resilience"
date: 2026-07-14
categories: [Homelab, Active Directory]
tags: [gpo, security, active-directory]
hidden: true
sitemap: false
permalink: /posts/ad-phase-4-resilience/
---

Having a single Domain Controller running in a lab is great for learning the basics, but in a real enterprise environment, relying on one DC is a huge gamble. If that single server goes down for maintenance, crashes, or suffers a hardware failure, your entire network loses authentication, DNS, and access to domain resources.

Phase 4 is all about turning my Active Directory environment into a highly available, fault-tolerant enterprise network. I focused on removing single points of failure, protecting core database files, and ensuring the domain stays resilient against unexpected downtime.

---

## **Step 1: Secondary Domain Controller Deployment (High Availability)**

To ensure continuous domain availability, I brought a second server online (**`NYCE-DC02`**) and promoted it as a Secondary Domain Controller to run alongside **`NYCE-DC01`**.

### <span style="color: #4A90E2;">1. Pre-Requisites & Network Setup</span>
Before promoting the server, I configured static network parameters on `NYCE-DC02` so it could talk directly to the primary domain controller:
* **Static IP Address:** `192.168.1.101/24`
* **Preferred DNS:** `192.168.1.100` (Points directly to `NYCE-DC01` for initial domain discovery)
* **Alternate DNS:** `127.0.0.1` (Self-referencing loopback address)

### <span style="color: #4A90E2;">2. Promotion & Active Directory Replication</span>
After installing the **Active Directory Domain Services (AD DS)** role on `NYCE-DC02`, I promoted it by joining it as an additional Domain Controller to the existing domain (`nycehomelab.local`). 

Once the promotion completed and the server rebooted, both Domain Controllers immediately began replicating directory data, DNS zones, and SYSVOL shares across the network.

### <span style="color: #4A90E2;">3. Verifying Replication & Health</span>
To verify that domain objects and schema changes were properly syncing between `NYCE-DC01` and `NYCE-DC02`, I ran the following built-in command-line tools:

<div class="callout callout-note">:: Checks the overall replication health across all Domain Controllers<p style="margin-top: 0px; margin-bottom: 0;">
<strong>repadmin /replsummary</strong></p>
</div>
<div class="callout callout-note">:: Performs a detailed check on inbound replication neighbors<p style="margin-top: 0px; margin-bottom: 0;">
<strong>repadmin /showrepl</strong></p>
</div>

---

## **Step 2: Backup, SYSVOL & Active Directory Disaster Recovery**

With two Domain Controllers providing high availability, the next critical step was disaster recovery. Redundancy protects against server failure, but it doesn't protect against corrupted database files, ransomware, or accidental mass object deletions.

To safeguard the environment, I configured built-in backup tools and recovery features to protect the core Active Directory database (`NTDS.dit`) and SYSVOL share.

---

### <span style="color: #4A90E2;">1. Enabling the Active Directory Recycle Bin</span>
By default, deleting an object in Active Directory (like a user account or OU) marks it as tombstoned, making instant restoration difficult. Enabling the Active Directory Recycle Bin allows deleted objects to be restored instantly with all their attributes (SID, group memberships, passwords) completely intact.

I enabled the Recycle Bin domain-wide via Active Directory Administrative Center (ADAC) and verified it using PowerShell:

```powershell
:: Enable Active Directory Recycle Bin for the domain
Enable-ADOptionalFeature -Identity 'Recycle Bin Feature' -Scope ForestOrConfigurationSet -Target 'nycehomelab.local' -Confirm:$false
```

<div class="callout callout-important"><strong>Testing Object Recovery:</strong><p style="margin-top: 10px; margin-bottom: 0;">I created a test user, deleted it, and restored it within seconds using Restore-ADObject without needing to reboot the Domain Controller into Directory Services Restore Mode (DSRM)!</p>
</div>

I created a test user, deleted it, and restored it within seconds using Restore-ADObject without needing to reboot the Domain Controller into Directory Services Restore Mode (DSRM)!


### <span style="color: #4A90E2;">2. System State Backups via Windows Server Backup</span>
Active Directory data cannot be backed up like regular files because the database files are constantly open and in use by the OS. I installed the Windows Server Backup feature on `NYCE-DC01` to capture a full System State Backup.

A System State backup includes:
* Active Directory Database (`NTDS.dit`)
* SYSVOL Folder Structure (Group Policies & Scripts)
* Registry, Boot Files, & System Volume
* DNS Server Data