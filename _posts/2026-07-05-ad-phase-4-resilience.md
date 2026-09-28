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

Phase 4 is all about turning my Active Directory environment into a **highly available, fault-tolerant enterprise network**. I focused on removing single points of failure, protecting core database files, and ensuring the domain stays resilient against unexpected downtime.

## **Step 1: Secondary Domain Controller Deployment (High Availability)**

To ensure continuous domain availability, I brought a second server online (**`NYCE-DC02`**) and promoted it as a Secondary Domain Controller to run alongside **`NYCE-DC01`**.

### <span style="color: #4A90E2;">1. Pre-Requisites & Network Setup
Before promoting the server, I configured static network parameters on `NYCE-DC02` so it could talk directly to the primary domain controller:

* **Static IP Address:** **`192.168.1.110/24`**
* **Preferred DNS:** **`192.168.1.100`** (Points directly to **`NYCE-DC01`** for initial domain discovery)
* **Alternate DNS:** `127.0.0.1` (Self-referencing loopback address)

### <span style="color: #4A90E2;">2. Promotion & Active Directory Replication
After installing the **Active Directory Domain Services (AD DS)** role on `NYCE-DC02`, I promoted it by joining it as an additional Domain Controller to the existing domain (**`nycehomelab.local`**). 

Once the promotion completed and the server rebooted, both Domain Controllers immediately began replicating directory data, DNS zones, and SYSVOL shares across the network.

### <span style="color: #4A90E2;">3. Verifying Replication & Health
To verify that domain objects and schema changes were properly syncing between **`NYCE-DC01`** and **`NYCE-DC02`**, I ran the following built-in command-line tools:

* **Active Directory Users and Computers (ADUC):** Right after promoting `NYCE-DC02`, opening **`dsa.msc`** confirmed that all previously created Organizational Units (OUs), security groups, and user accounts from `NYCE-DC01` automatically reflected without any manual copying or configuration.
* **Replication Diagnostics via CLI:** To confirm health status at the network layer, I ran the following commands:

    <div class="callout callout-note">:: Checks the overall replication health across all Domain Controllers<p style="margin-top: 0px; margin-bottom: 0;">
    <strong>repadmin /replsummary</strong></p>
    </div>
    <div class="callout callout-note">:: Performs a detailed check on inbound replication neighbors<p style="margin-top: 0px; margin-bottom: 0;">
    <strong>repadmin /showrepl</strong></p>
    </div>

<iframe width="100%" height="450" src="https://www.youtube.com/embed/ffxAKkCQ20Y?si=Urlm8Eoc62OILctV" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 2: Backup, SYSVOL & Active Directory Disaster Recovery**

With two Domain Controllers providing high availability, the next critical step was disaster recovery. Redundancy protects against server failure, but it doesn't protect against corrupted database files, ransomware, or accidental mass object deletions.

To safeguard the environment, I configured built-in backup tools and recovery features to protect the core Active Directory database (**`NTDS.dit`**) and SYSVOL share.

### <span style="color: #4A90E2;">1. Enabling Recycle Bin
By default, deleting an object in Active Directory (like a user account or OU) marks it as tombstoned, making instant restoration difficult. Enabling the Active Directory Recycle Bin ***allows deleted objects to be restored instantly with all their attributes*** (SID, group memberships, passwords) completely intact.

I enabled the Recycle Bin domain-wide via Active Directory Administrative Center (ADAC) and verified it using PowerShell:

```powershell
:: Enable Active Directory Recycle Bin for the domain
Enable-ADOptionalFeature -Identity 'Recycle Bin Feature' -Scope ForestOrConfigurationSet -Target 'nycehomelab.local' -Confirm:$false
```

<iframe width="100%" height="450" src="https://www.youtube.com/embed/-Q_hlkk4hD0?si=As6p0wlJkqpnqI5x" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### <span style="color: #4A90E2;">2. System State Backups via Windows Server Backup
Active Directory data cannot be backed up like regular files because the database files are constantly open and in use by the OS. I installed the Windows Server Backup feature on `NYCE-DC01` to capture a full System State Backup.

A System State backup includes:
* Active Directory Database (`NTDS.dit`)
* SYSVOL Folder Structure (Group Policies & Scripts)
* Registry, Boot Files, & System Volume
* DNS Server Data

<iframe width="100%" height="450" src="https://www.youtube.com/embed/dGvy6RddlCg?si=XBlJ92AzgIgk-Vsp" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

### <span style="color: #4A90E2;">3. Disaster Recovery Scenarios & Best Practices
I documented two distinct restoration approaches depending on the failure type:
* **Non-Authoritative Restore:** Used when a single DC crashes. You restore the System State, and the DC updates itself by pulling the latest active data from surviving Domain Controllers (like NYCE-DC02).

* **Authoritative Restore:** Used if data is accidentally deleted domain-wide (and bypassed the Recycle Bin). You restore the System State in DSRM mode and use ntdsutil to mark specific objects as authoritative, forcing them to replicate back out to all other DCs.

<div class="callout callout-important"><strong>Key Takeaway:</strong><p style="margin-top: 10px; margin-bottom: 0;">High availability keeps the network running, but solid backups ensure you can recover when things go totally wrong. Combining the AD Recycle Bin with regular System State backups gives the lab complete data resilience.</p>
</div>


## **Step 3: Centralized Security Auditing & Event Logging**

Once high availability and backups were secured, the next logical step was visibility. In an enterprise network, you can't protect what you can't see. Without proper auditing, unauthorized privilege changes, account lockouts, or suspicious logons go completely unnoticed.

In this step, I configured Advanced Audit Policies across the domain to ensure critical security events are logged, tracked, and ready for analysis.

---

### <span style="color: #4A90E2;">1. Configuring Advanced Audit Policies
Instead of using basic legacy auditing, I used **Advanced Audit Policy Configuration** via Group Policy (`COMP-Audit_Logging`) to capture specific, high-fidelity security events without cluttering the logs with noise.

Key audit subcategories configured:
* **Account Management:** Tracks user creation, deletion, and group membership changes.
* **Logon/Logoff:** Tracks interactive logons, network authentication, and failed attempts.
* **Directory Service Access:** Tracks changes made directly to Active Directory objects.
* **Privilege Use:** Tracks when administrative rights or sensitive privileges are exercised.

---

### <span style="color: #4A90E2;">2. Key Security Event IDs Monitored
I documented the essential Event IDs every sysadmin and SOC analyst needs to watch inside Windows Event Viewer (`Security` log):

| Event ID | Event Type | Description / Security Context |
| :--- | :--- | :--- |
| **4624** | Successful Logon | Confirms user authentication and identifies the logon type (e.g., Type 2 Interactive, Type 10 RDP). |
| **4625** | Failed Logon | Crucial for detecting brute-force attacks or incorrect password configurations. |
| **4720** | User Account Created | Alerts when a new user profile is created in Active Directory. |
| **4728** | Member Added to Group | Tracks when a user is added to a sensitive security group (e.g., `Domain Admins`). |
| **4740** | Account Locked Out | Identifies when an account is locked out due to repeated failed logon attempts. |
| **5136** | Directory Object Modified | Logs attribute-level changes made to AD objects for change management. |

---

### <span style="color: #4A90E2;">3. Testing Audit Logging in the Lab
To test my auditing setup:
1. Created a standard user account on `NYCE-DC01` and added it to a local group.
2. Verified that Event IDs **4720** and **4728** generated immediately in the Security event log with full details (showing *who* made the change, *when*, and *what* account was modified).
3. Intentionally entered wrong passwords on `CLIENT01` to confirm Event ID **4625** correctly captured the source IP and target username.

<div class="callout callout-important"><strong>Key Takeaway:</strong><p style="margin-top: 0px; margin-bottom: 0;">
Proper event auditing turns Active Directory from a black box into a fully transparent environment. Having these logs active is the first step toward integrating with a SIEM tool (like Microsoft Sentinel or Splunk) in the future!</p>
</div>

## **Step 4: Core Network Resilience (DHCP Failover & Scope Redundancy)**

Having two Domain Controllers replicating directory data and DNS was a huge milestone, but I realized client PCs still had a single point of failure: DHCP. If the single server handing out IP addresses went down, new devices wouldn't be able to get online, and existing devices would lose connection as soon as their leases expired.

To make the lab network truly resilient, I decided to set up high-availability **DHCP Failover** between `NYCE-DC01` and `NYCE-DC02`

---

### <span style="color: #4A90E2;">1. Installing DHCP & Authorizing the Secondary Server
I wanted to practice two different ways to deploy roles in Windows Server, so I used the traditional GUI on the primary server and PowerShell on the secondary one:

* **Primary Server (NYCE-DC01):** I installed the DHCP Server role using the **Server Manager GUI** (`Add Roles and Features Wizard`) and set up the initial IPv4 scope.

<iframe width="100%" height="450" src="https://www.youtube.com/embed/0Y9DuZmjJ84?si=C9mRmGpCEmKpLK7m" title="Installing DHCP Server Role via GUI" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

* **Secondary Server (NYCE-DC02):** To speed things up, I jumped straight into PowerShell on DC02 and installed the role with a single command:

```powershell
    Install-WindowsFeature -Name DHCP -IncludeManagementTools
```

<iframe width="100%" height="450" src="https://www.youtube.com/embed/Wo0NsmCf1rk?si=8iQa-c3EBE5KzNqD" title="Installing DHCP Server Role via Powershell" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

* **Double-Checking My Work:** I opened the DHCP Console (dhcpmgmt.msc) on DC01 and verified that both NYCE-DC01 and NYCE-DC02 showed up as trusted, authorized servers in Active Directory

---

### <span style="color: #4A90E2;">2. Configuring DHCP Failover Mode
Instead of setting up a complex split-scope configuration, I used native Windows Server **DHCP Failover** directly on the main scope (**`192.168.10.0/24`**). 

I evaluated two failover modes:
* **Load Balance Mode (50/50):** Both servers actively handle client requests simultaneously, splitting the IP pool address load evenly.
* **Hot Standby Mode:** The primary server (`NYCE-DC01`) handles 100% of the traffic, while the secondary server (`NYCE-DC02`) stays on standby, taking over only if the primary fails.

I selected ***Hot Standby Mode*** with the following parameters:
* **Partner Server:** **`NYCE-DC02.nycehomelab.local`**
* **Role:** Active (`NYCE-DC01`) / Standby (`NYCE-DC02`)
* **Reserve Address Percentage:** `5%` (Allocated for standby leases during failover)
* **State Switchover Interval (MCLT):** `60 minutes` (Automatic failover delay threshold)
* **Shared Secret:** Encrypted authentication key between both DHCP servers.

<iframe width="100%" height="450" src="https://www.youtube.com/embed/1qFyKjncRzA?si=Hed5jpDrBZ7-ykrZ" title="Configuring DHCP Failover Mode" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

### <span style="color: #4A90E2;">3. Failover Troubleshooting "Lost Contact with Partner"
Right after set up the failover, I hit an unexpected roadblock: **both Domain Controllers showed a state of *"Lost contact with partner"***.

Here is how I investigated and resolved the issue step-by-step:

* **Step A: Unblocking Firewall Rules**

    I suspected that Windows Firewall might be blocking the failover communication traffic **`(UDP Port 647)`**. To rule this out, I ran this PowerShell command on both `NYCE-DC01` and `NYCE-DC02` to allow all DHCP-related traffic:

    ```powershell
        Enable-NetFirewallRule -DisplayGroup "DHCP Server"
    ```

* **Step B: Spotting the Configuration Misstep**
    
    Even after enabling the firewall rules, the status remained stuck on "Lost contact with partner". I opened the scope properties and looked closer at the **Failover tab**. That’s when I noticed the issue: the **Partner Server was incorrectly pointing back to NYCE-DC01 instead of NYCE-DC02!** The server was trying to establish a failover relationship with itself.

* **Step C: Re-configuring the Relationship**
    
    To fix this loop, I re-did the configuration properly: Right-clicked the scope on NYCE-DC01 and selected **Deconfigure Failover**. Right-clicked the scope again and selected **Configure Failover**....
    In the wizard, instead of typing the name manually, I selected NYCE-DC02 directly from the list of **authorized Active Directory DHCP servers.**

    After reconfiguration, both servers successfully communicated and updated their status to Normal!

<iframe width="100%" height="450" src="https://www.youtube.com/embed/D8STyFE6aRs?si=JeWneJybDL9WZI3o" title="Failover TS   Lost Contact with Partner" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>







---

> **Phase 4 Summary & Final Thoughts:**  
> With Phase 4 complete, my homelab has evolved into a resilient, enterprise-grade architecture. By adding a Secondary Domain Controller, setting up AD Recycle Bin and System State backups, enforcing Advanced Security Auditing, deploying an internal PKI, and securing DHCP with Hot Standby failover, the environment is fully protected against single points of failure!