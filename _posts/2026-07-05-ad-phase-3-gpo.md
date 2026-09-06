---
title: "Phase 3: Group Policy Governance & System Hardening"
date: 2026-07-10
categories: [Homelab, Active Directory]
tags: [gpo, gpmc, laps, security-hardening, active-directory]
hidden: true
sitemap: false
permalink: /posts/ad-phase-3-gpo/
---

Now that my Active Directory foundation was solid and my user hierarchy was organized, it was time for the fun part: **locking things down and automating my environment**. 

Phase 3 is all about **Group Policy Objects (GPOs)**. Early on, I realized how easy it is to fall into the trap of shoving dozens of unrelated settings into a single, massive GPO—making it a complete nightmare to manage or troubleshoot later. 

To keep my homelab clean, scalable, and built like a real production network, I adopted a **Single-Purpose (Modular) GPO Architecture**. Instead of one giant policy, I broke everything down into focused, bite-sized GPOs with clear naming conventions:
* **`COMP-`** for machine/hardware-level configurations (applies on system startup).
* **`USER-`** for account-level configurations (applies when a user logs in).

Here is how I set it all up step-by-step!

## **Step 1: Centralizing Policy Management (ADMX Central Store)**
When managing GPOs across a domain, relying on local template files (`.admx`) stored on individual PCs can quickly get messy. If another admin (or even myself on a different machine) edits a GPO using an older Windows version, settings can easily get overwritten or missed altogether.

To keep everything consistent across the domain, I set up a **Central Store** on my Domain Controller (**`NYCE-DC01`**). This forces Group Policy to pull its template definitions from a single, shared folder in **`SYSVOL`** instead of local storage.

<div class="callout callout-note"><strong>NOTE:</strong><p style="margin-top: 10px; margin-bottom: 0;">If you want to manage features specific to newer Windows builds (like Windows 11 23H2/24H2), just extract the updated <strong>.admx</strong> files into this SYSVOL folder. It instantly updates the available GPO settings for all admins across the domain!.</p>
</div>

**Any updates made to `.admx` files in `SYSVOL` will automatically apply domain-wide for anyone editing GPOs!**

<iframe width="100%" height="450" src="https://www.youtube.com/embed/BtuXd0MRybI?si=ZgJ2DW5wyfyw0-Gm" title="ADMX Central Store" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## **Step 2: Understanding GPO Structure & Enforcement**
Once the Central Store was up and running, it was time to establish a baseline security policy for my workstations. But before diving into the Group Policy editor, I had to get a clear handle on how GPO settings are structured and where they belong.

### <span style="color: #4A90E2;">A. Computer Configuration vs. User Configuration</span>
* **Computer Configuration:** Applies directly to the machine itself, regardless of who signs in. It processes at system startup (e.g., security hardening, firewall rules, local account controls).
* **User Configuration:** Applies to the user account, following that person no matter which domain machine they log into (e.g., desktop wallpaper, drive mappings, browser settings).

### <span style="color: #4A90E2;">B. Policies vs. Preferences</span>
* **Policies (Enforced Rules):** Think of this as a strict workplace rule, like wearing a mandatory security badge. It is non-negotiable, locked down, and employees cannot change or turn it off.
* **Preferences (Flexible Defaults):** Think of this as the company handing a new employee a desk setup on Day 1. This is the initial setup. They set up your monitor height and give you a default penholder for convenience, but if you want to move the penholder to the left side of your desk, you’re free to do so.

# **Step 3: Refining Core Authentication (Default Domain Policy)**
   <div class="callout callout-danger"><strong>WARNING:</strong>
   <p style="margin-top: 10px; line-height: 1.6;">
   <strong>Do not modify the Default Domain Policy.</strong> This GPO is linked directly to the root of the domain, meaning every single user and computer processes it.
   </p>
   <p style="line-height: 1.6;">
    Because of that, it should only be used for four specific areas: <strong><em>account policy settings, password policy, account lockout policy, and Kerberos policy</em></strong>.
   </p>
   <p style="margin-bottom: 0; line-height: 1.6;">
    Any other setting—like software installs, desktop configurations, or firewall rules—should go into a separate, focused GPO. Keeping GPOs small and modular makes management so much easier.
   </p>
   </div>

With that in mind, here are the only **baseline authentication rules** I configured inside the **`Default Domain Policy:`**

   | Policy Setting | Configuration | Purpose / Enterprise Context |
   | :--- |  :--- | :--- |
   | **Account Lockout Threshold** | **5 invalid attempts** | Aligns with CIS Benchmarks to prevent brute-force attacks while mitigating Denial of Service (DoS) risk. |
   | **Account Lockout Duration** | **30 minutes** | Implements an automatic cooldown period without requiring manual IT Helpdesk intervention. |
   | **Reset Lockout Counter After** | **30 minutes** | Sets the observation window before resetting the failed attempts counter back to zero. |
   | **Minimum Password Length** | **12 characters** | Replaces legacy 8-character limits to defend against modern GPU-based password hashing tools. |
   | **Password Complexity** | **Enabled** | Enforces 3 of 4 character classes (uppercase, lowercase, numbers, special characters). |
   | **Enforce Password History** | **24 passwords** | Prevents users from immediately cycling back to previous passwords. |

   <iframe width="100%" height="450" src="https://www.youtube.com/embed/cL6YpH2hE4c?si=y5nCyX1JUzXVe6i1" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## **Step 4: Building Modular GPOs (The Single-Purpose Approach)**
Instead of creating one monolithic "Workstation Policy," I split my configurations into **dedicated, modular GPOs**. This modular setup makes it super easy to isolate issues—if drive mappings stop working, I only need to inspect or toggle the Drive Mapping GPO without touching firewall or security settings!

Here are the specific, modular GPOs I created for my endpoints:

### A. Computer Hardening & System Policies

* **`COMP-Interactive_Logon_Banner`**
   * *Configuration:* Local Policies ➔ Security Options ➔ Interactive logon: Message title... ("WARNING")
   and Interactive logon: Message text... ("This system is restricted to authorized NYCE Home Lab users.").
   * *Purpose:* Enforces a mandatory legal notification banner prior to the Windows logon prompt for audit compliance.

* **`COMP-Audit_Logging`**  
  * *Configuration:* Advanced Audit Policy ➔ Logon/Logoff ➔ Audit Success & Failure.  
  * *Purpose:* Generates Event IDs 4624/4625 for SIEM and SOC monitoring.
* **`COMP-Disable_Guest_Account`**  
  * *Configuration:* Local Policies ➔ Security Options ➔ Accounts: Guest account status ➔ **Disabled**.  
  * *Purpose:* Closes unauthenticated local entry points across all endpoints.
* **`COMP-Prevent_LAN_Manager_Hash`**  
  * *Configuration:* Security Options ➔ Network security: Do not store LAN Manager hash value on next password change ➔ **Enabled**.  
  * *Purpose:* Stops caching vulnerable LM hashes in RAM/LSASS.
* **`COMP-Restrict_Blank_Password_Console`**  
  * *Configuration:* Security Options ➔ Accounts: Limit local account use of blank passwords to console logon only ➔ **Enabled**.  
  * *Purpose:* Blocks network access to local accounts without passwords.
* **`COMP-Disable_Forced_Restarts`**  
  * *Configuration:* Windows Components ➔ Windows Update ➔ No auto-restart with logged on users for scheduled updates ➔ **Enabled**.  
  * *Purpose:* Prevents unsaved work loss during patch deployments.
* **`COMP-Audit_GPO_Changes`**  
  * *Configuration:* Advanced Audit Policy ➔ DS Access ➔ Audit Directory Service Changes ➔ **Success & Failure**.  
  * *Purpose:* Logs GPO setting modifications (Event ID 5136) for change tracking.
* **`COMP-Block_Microsoft_Store`**  
  * *Configuration:* Windows Components ➔ Store ➔ Turn off the Store application ➔ **Enabled**.  
  * *Purpose:* Prevents employees from installing unapproved games or apps.
* **`COMP-Disable_Anonymous_SID_Translation`**  
  * *Configuration:* Security Options ➔ Network access: Allow anonymous SID/Name translation ➔ **Disabled**.  
  * *Purpose:* Prevents attackers from enumerating domain account usernames via SIDs.
* **`COMP-Restrict_Anonymous_Permissions`**  
  * *Configuration:* Security Options ➔ Network access: Let Everyone permissions apply to anonymous users ➔ **Disabled**.  
  * *Purpose:* Restricts anonymous network shares enumeration.
* **`COMP-Audit_NTLM_Usage`**  
  * *Configuration:* Security Options ➔ Network security: Restrict NTLM: Audit NTLM authentication in this domain ➔ **Enable all**.  
  * *Purpose:* Tracks legacy NTLM authentication before phasing it out for Kerberos.
* **`COMP-Disable_LLMNR`**  
  * *Configuration:* Network ➔ DNS Client ➔ Turn off multicast name resolution ➔ **Enabled**.  
  * *Purpose:* Mitigates LLMNR/NBT-NS credential poisoning attacks (e.g., Responder tools).
* **`COMP-Control_Local_Admins_Group`**  
  * *Configuration:* Preferences ➔ Local Users and Groups ➔ Local Group (Administrators) ➔ Update/Remove unauthorized users.  
  * *Purpose:* Strips local administrator privileges from standard users.
* **`COMP-Windows_Firewall_Rules`**  
  * *Configuration:* Security Settings ➔ Windows Defender Firewall ➔ Enforce default-block inbound, allow ICMP Ping & WinRM.  
  * *Purpose:* Secures network boundaries while allowing centralized monitoring.
* **`COMP-Enable_UAC`**  
  * *Configuration:* Security Options ➔ User Account Control: Run all administrators in Admin Approval Mode ➔ **Enabled**.  
  * *Purpose:* Ensures privilege elevation prompts are enforced even for admins.
* **`COMP-AppLocker_Execution_Rules`**  
  * *Configuration:* Security Settings ➔ Application Control Policies ➔ AppLocker ➔ Restrict binaries to `%ProgramFiles%` and `%SystemRoot%`.  
  * *Purpose:* Blocks execution of malicious `.exe`/`.ps1` scripts in user-writable paths (`%AppData%`, `%Temp%`).
* **`COMP-Block_Removable_Media`**  
  * *Configuration:* Administrative Templates ➔ System ➔ Removable Storage Access ➔ All Removable Storage classes: Deny all access ➔ **Enabled**.  
  * *Purpose:* Blocks USB drives and external storage to prevent malware infection and data exfiltration.

---

### B. User Workspace & Environment Policies

* **`USER-ALL-SEC-Workstation_Restrictions`**
  * *Prohibit Access to Control Panel & Settings:* **Enabled** (Prevents standard staff from messing with network adapters).
  * *Prevent Access to Registry Editing Tools (`regedit`):* **Enabled** (Blocks unauthorized registry tweaks and script executions).

* **`USER-ALL-CFG-Screen_Lock_Timeout`**
  * *Enable Screen Saver:* **Enabled**
  * *Password Protect Screen Saver:* **Enabled**
  * *Screen Saver Timeout:* **600 seconds (10 mins)** (Automatically locks unattended workstations).

* **`USER-HR-CFG-Automated_Drive_Mappings`**
  * *Item-Level Targeting:* Automatically maps `\\NYCE-DC01\HRUsers$` as the `S:\` drive upon login, strictly for users in the HR Organizational Unit.

* **`USER-Screen_Lock_Timeout`**  
  * *Configuration:* Control Panel ➔ Personalization ➔ Screen saver timeout (600s / 10 mins) & Password protect ➔ **Enabled**.  
  * *Purpose:* Prevents physical unauthorized access to unattended endpoints.
* **`USER-Restrict_CMD_PowerShell`**  
  * *Configuration:* System ➔ Prevent access to the command prompt ➔ **Enabled** (Disables script execution).  
  * *Purpose:* Blocks standard users from running command-line tools.
* **`USER-Restrict_Control_Panel`**  
  * *Configuration:* Control Panel ➔ Prohibit access to Control Panel and PC settings ➔ **Enabled**.  
  * *Purpose:* Prevents standard users from modifying adapter settings or system configurations.
* **`USER-Restrict_Software_Installation`**  
  * *Configuration:* Windows Components ➔ Windows Installer ➔ Turn off Windows Installer ➔ **Enabled**.  
  * *Purpose:* Blocks standard accounts from installing `.msi` software packages.
* **`USER-Restrict_Registry_Tools`**  
  * *Configuration:* System ➔ Prevent access to registry editing tools (`regedit`) ➔ **Enabled**.  
  * *Purpose:* Blocks users from manually altering system keys.
* **`USER-Automated_Drive_Mappings`**  
  * *Configuration:* Preferences ➔ Windows Settings ➔ Drive Maps ➔ Item-Level Targeting for HR OU (`S:\` Drive).  
  * *Purpose:* Automatically maps network file shares based on user department.
* **`USER-Default_Printers_Deployment`**  
  * *Configuration:* Preferences ➔ Control Panel Settings ➔ Printers ➔ Shared Printer deployment with Item-Level Targeting.  
  * *Purpose:* Connects users to the correct network printers automatically upon login.


---

## **Step 5. GPO Node Optimization (Disabling Unused Settings)**

When a GPO contains only machine or user rules, the unused section should be explicitly disabled via **GPMC > Details > GPO Status**:

* **User Configuration Settings Disabled:** Forces Windows to completely skip searching for user-based rules when processing a machine GPO (like `C-Workstation-Baseline`).
* **Computer Configuration Settings Disabled:** Forces Windows to bypass hardware/OS rule processing when evaluating a user GPO (like `U-Workstation-Baseline`).

<div class="callout callout-important"><strong>Why This Matters:</strong><p style="margin-top: 10px; margin-bottom: 0;">Disabling unused nodes <strong>speeds up client startup and logon times</strong>, reduces Domain Controller processing overhead, and keeps policy troubleshooting clean and modular.</p>
</div>

---

**What's Next?**  
*Up Next: In **Phase 4: Group Policy & Advanced Security (GPO)**, we will automate configurations, map these shared network drives automatically for our users upon login, and enforce security baselines across the domain workstations.*



