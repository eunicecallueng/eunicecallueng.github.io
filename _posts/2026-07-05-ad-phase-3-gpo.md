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

## **Step 3: Refining Core Authentication (Default Domain Policy)**
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

   <iframe width="100%" height="450" src="https://www.youtube.com/embed/cL6YpH2hE4c?si=y5nCyX1JUzXVe6i1" title="Refining Core Authentication (Default Domain Policy)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## **Step 4: Building Modular GPOs (The Single-Purpose Approach)**
Instead of creating one monolithic "Workstation Policy," I split my configurations into **dedicated, modular GPOs** using the **`[SCOPE]-[PURPOSE]`** naming convention. This modular setup makes it super easy to isolate issues—if drive mappings stop working, I only need to inspect or toggle the Drive Mapping GPO without touching firewall or security settings!

Here are the specific, modular GPOs I created for my endpoints:

### <span style="color: #4A90E2;">A. Computer Hardening & System Policies</span>

| GPO Name | Purpose / Enterprise Context | Core Configuration  |
| :--- | :--- | :--- |
| **`COMP-Interactive_Logon_Banner`** | Enforces mandatory legal notifications prior to logon prompt for compliance. | Local Policies ➔ Security Options ➔ Interactive logon: Message title & text configured |
| **`COMP-Audit_Logging`** | Generates Event IDs 4624/4625 for SIEM and SOC monitoring. | Advanced Audit Policy ➔ Logon/Logoff ➔ Audit Success & Failure |
| **`COMP-Disable_Guest_Account`** | Closes unauthenticated local entry points across all endpoints. | Local Policies ➔ Security Options ➔ Accounts: Guest account status ➔ **Disabled** |
| **`COMP-Block_Removable_Media`** | Blocks USB drives and external storage to prevent malware infection and data exfiltration. | Administrative Templates ➔ System ➔ Removable Storage Access ➔ All Removable Storage classes: Deny all access ➔ **Enabled** |
| **`USER-Prohibit_User_Installs`** | Blocks standard accounts from installing `.msi` software packages. | Administrative Template ➔ Windows Components ➔ Windows Installer ➔ Prohibit User Installs ➔ **Enabled** | 
| **`COMP-Prevent_LAN_Manager_Hash`** | Stops caching vulnerable LM hashes in RAM/LSASS. | Security Options ➔ Network security: Do not store LAN Manager hash value on next password change ➔ **Enabled** |
| **`COMP-Restrict_Blank_Password_Console`** | Blocks network access to local accounts without passwords. | Security Options ➔ Accounts: Limit local account use of blank passwords to console logon only ➔ **Enabled** |
| **`COMP-Disable_Forced_Restarts`** | Prevents unsaved work loss during patch deployments. | Windows Components ➔ Windows Update ➔ No auto-restart with logged on users for scheduled updates ➔ **Enabled** |
| **`COMP-Audit_GPO_Changes`** | Logs GPO setting modifications (Event ID 5136) for change tracking. | Advanced Audit Policy ➔ DS Access ➔ Audit Directory Service Changes ➔ **Success & Failure** |
| **`COMP-Block_Microsoft_Store`** | Prevents employees from installing unapproved games or apps. | Windows Components ➔ Store ➔ Turn off the Store application ➔ **Enabled** | 
| **`COMP-Disable_Anonymous_SID_Translation`** | Prevents attackers from enumerating domain account usernames via SIDs. | Security Options ➔ Network access: Allow anonymous SID/Name translation ➔ **Disabled** |
| **`COMP-Restrict_Anonymous_Permissions`** | Restricts anonymous network shares enumeration. | Security Options ➔ Network access: Let Everyone permissions apply to anonymous users ➔ **Disabled** |
| **`COMP-Audit_NTLM_Usage`** | Tracks legacy NTLM authentication before phasing it out for Kerberos. | Security Options ➔ Network security: Restrict NTLM: Audit NTLM authentication in this domain ➔ **Enable all** |
| **`COMP-Disable_LLMNR`** | Mitigates LLMNR/NBT-NS credential poisoning attacks (e.g., Responder tools). | Network ➔ DNS Client ➔ Turn off multicast name resolution ➔ **Enabled** | 
| **`COMP-Control_Local_Admins_Group`** | Strips local administrator privileges from standard users. | Preferences ➔ Local Users and Groups ➔ Local Group (Administrators) ➔ Update/Remove unauthorized users | 
| **`COMP-Windows_Firewall_Rules`** | Secures network boundaries while allowing centralized monitoring. | Security Settings ➔ Windows Defender Firewall ➔ Enforce default-block inbound, allow ICMP Ping & WinRM | 
| **`COMP-Enable_UAC`** | Ensures privilege elevation prompts are enforced even for admins. | Security Options ➔ User Account Control: Run all administrators in Admin Approval Mode ➔ **Enabled** | 
| **`COMP-AppLocker_Execution_Rules`** | Blocks execution of malicious `.exe`/`.ps1` scripts in user-writable paths (`%AppData%`, `%Temp%`). | Security Settings ➔ Application Control Policies ➔ AppLocker ➔ Restrict binaries to `%ProgramFiles%` and `%SystemRoot%` |

<iframe width="100%" height="450" src="https://www.youtube.com/embed/R0Q8WoLOuOc?si=e2YIEAqocEqOgObP" title="Computer Hardening & System Policies" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


---

### <span style="color: #4A90E2;">B. User Workspace & Environment Policies</span>

| GPO Name | Purpose / Enterprise Context | Core Configuration | 
| :--- | :--- | :--- |
| **`USER-Screen_Lock_Timeout`** | Prevents physical unauthorized access to unattended endpoints. | Control Panel ➔ Personalization ➔ Screen saver timeout (600s / 10 mins) & Password protect ➔ **Enabled** | 
| **`USER-Restrict_CMD_PowerShell`** | Blocks standard users from running command-line tools. | System ➔ Prevent access to the command prompt ➔ **Enabled** (Disables script execution) | 
| **`USER-Restrict_Control_Panel`** | Prevents standard users from modifying adapter settings or system configurations. | Control Panel ➔ Prohibit access to Control Panel and PC settings ➔ **Enabled** | 
| **`USER-Restrict_Registry_Tools`** | Blocks users from manually altering system keys. | System ➔ Prevent access to registry editing tools (`regedit`) ➔ **Enabled** | 
| **`USER-Automated_Drive_Mappings`** | Automatically maps network file shares based on user department. | Preferences ➔ Windows Settings ➔ Drive Maps ➔ Item-Level Targeting for HR OU (`S:\` Drive) | 
| **`USER-Default_Printers_Deployment`** | Connects users to the correct network printers automatically upon login. | Preferences ➔ Control Panel Settings ➔ Printers ➔ Shared Printer deployment with Item-Level Targeting | 

<iframe width="100%" height="450" src="https://www.youtube.com/embed/yRq__B3U33E?si=QeY91SezIcDeTklQ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<iframe width="100%" height="450" src="https://www.youtube.com/embed/lyBGzIYnI4E?si=FoXsskPgIy1Uh2k_" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
---

### <span style="color: #4A90E2;">C. GPO Node Optimization (Disabling Unused Settings)</span>

When a GPO contains only machine or user rules, the unused section should be explicitly disabled via **GPMC > Details > GPO Status**:

* **User Configuration Settings Disabled:** Forces Windows to completely skip searching for user-based rules when processing a machine GPO (like `C-Workstation-Baseline`).
* **Computer Configuration Settings Disabled:** Forces Windows to bypass hardware/OS rule processing when evaluating a user GPO (like `U-Workstation-Baseline`).

<div class="callout callout-important"><strong>Why This Matters:</strong><p style="margin-top: 10px; margin-bottom: 0;">Disabling unused nodes <strong>speeds up client startup and logon times</strong>, reduces Domain Controller processing overhead, and keeps policy troubleshooting clean and modular.</p>
</div>

---

## **Step 5: Verification & Troubleshooting**

Setting up Group Policies in the console is super satisfying, but the real test is seeing if they actually kick in on the client PC without breaking anything!

I learned pretty quickly that you shouldn't just assume a policy worked just because you clicked "Apply." To make sure my security tweaks and drive mappings were actually doing their job, I built a quick routine to test, verify, and troubleshoot my setup.

---

### <span style="color: #4A90E2;">A. Testing Policies on the Client (``gpupdate``)</span>

Normally, Windows takes its sweet time (***about 90 to 120 minutes***) to pull new Group Policies in the background. But when you’re actively testing in a lab, nobody has time to wait around for that!

To force my client PC to pick up new changes right away, I pop open the command prompt and run:

<div class="callout callout-note">:: Quick refresh for basic setting changes<p style="margin-top: 10px; margin-bottom: 0;">
<strong>gpupdate</strong></p>
</div>
<div class="callout callout-note">:: Forces a complete re-download of EVERY policy<p style="margin-top: 0px; margin-bottom: 0;">
<strong>gpupdate /force</strong></p>
</div>

<div class="callout callout-important"><strong>Note:</strong><p style="margin-top: 10px; margin-bottom: 0;">If you're testing things like Software Installs (.msi) or Folder Redirection, Windows usually can't apply them while you're actively logged in. Don't panic if it doesn't show up immediately—gpupdate /force will usually ask you to log off or reboot to finish the job!</p>
</div>

### <span style="color: #4A90E2;">B. Double-Checking What Applied (`gpresult`)</span>

When a policy isn't working, my go-to tool is **`gpresult`**. It gives you a clear breakdown of what policies were actually applied to the computer or user, and which ones were ignored.

#### 1. Quick Command-Line Check
If I just want a fast summary right inside the terminal:

<div class="callout callout-note">:: Shows all applied policies for the current user and PC<p style="margin-top: 0px; margin-bottom: 0;">
<strong>gpresult /r</strong></p>
</div>
<div class="callout callout-note">:: Focuses strictly on computer-level policies (run as Admin!)<p style="margin-top: 0px; margin-bottom: 0;">
<strong>gpresult /scope computer /r</strong></p>
</div>

#### 2. The Full Graphical Diagnostic Report
If I want to **dig deeper or save a snapshot of my setup**, I export an HTML report:
<div class="callout callout-note">gpresult /h C:\GPO_Report.html<p style="margin-top: 0px; margin-bottom: 0;"></p>
</div>

When I open that HTML file in a browser, I specifically look out for:
   * **Applied GPOs:** The policies that loaded successfully.
   * **Denied GPOs:** The ones that were blocked (and super helpfully, it tells you why—like permission issues or WMI filtering).

### <span style="color: #4A90E2;">C. Real Troubleshooting Scenarios I Ran Into</span>

Building this lab wasn't totally smooth sailing, but troubleshooting the hiccups was honestly where I learned the most! Here are a few real issues I ran into on my test client (`CLIENT01`):

#### 1. The Mysterious "Denied GPO (Security Filtering)"
* **The Issue:** I created a drive mapping GPO (`USER-Automated_Drive_Mappings`) for the HR team, but it refused to apply to my test user. `gpresult` marked it as *Denied*.
* **What Went Wrong:** When I removed `Authenticated Users` to target only the `SG-HR-Users` group, the client computer itself lost permission to read the policy settings from SYSVOL!
* **How I Fixed It:** In the GPO’s **Delegation** tab, I added `Domain Computers` with **Read** access. That let the PC read the policy while keeping the actual settings restricted strictly to the HR group.

#### 2. Fixing Slow Logon Times
* **The Issue:** Logging into the client machine started feeling a bit sluggish.
* **What Went Wrong:** My computer-only policies (like USB blocking) were still trying to look for user settings, wasting extra processing time.
* **How I Fixed It:** I went into GPMC, selected my computer GPOs, and set **User Configuration Settings Disabled**. This told Windows to completely skip checking user settings for those policies, making logons snappy again!

#### 3. Checking Event Viewer for Clues
When the command prompt doesn't give enough details, Windows Event Viewer has a dedicated log that tells you the exact story of what happened during policy processing:

* **Where to Look:** `Applications and Services Logs -> Microsoft -> Windows -> GroupPolicy -> Operational`
* **Key Event IDs to Remember:**
  * **Event 4001:** Group Policy started processing.
  * **Event 8001:** Success! Everything applied clean.
  * **Event 7016:** Network delay or couldn't reach the Domain Controller.

---
**What's Next?**  
*Up Next: In **Phase 4: Group Policy & Advanced Security (GPO)**, we will automate configurations, map these shared network drives automatically for our users upon login, and enforce security baselines across the domain workstations.*



