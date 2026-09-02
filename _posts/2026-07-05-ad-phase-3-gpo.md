---
title: "Phase 3: Group Policy Governance & System Hardening"
date: 2026-07-10
categories: [Homelab, Active Directory]
tags: [gpo, gpmc, laps, security-hardening, active-directory]
hidden: true
sitemap: false
permalink: /posts/ad-phase-3-gpo/
---

With Active Directory set up and my user hierarchy organized, it was time to lock things down. Phase 3 is all about Group Policy Objects (GPOs)—specifically, taking my homelab from default settings to proper security standards.

In this phase, I focused on centralizing policy management, enforcing baseline security controls, and sticking to modular GPO conventions (`C-` for computer policies and `U-` for user policies) so the environment stays clean, scalable, and easy to manage. Here is how I set it up step-by-step!

## **Step 1: Centralizing Policy Management (ADMX Central Store)**
When managing GPOs across a domain, relying on local template files (`.admx`) stored on individual PCs can quickly get messy. If another admin (or even myself on a different machine) edits a GPO using an older Windows version, settings can easily get overwritten or missed altogether.

To keep everything consistent across the domain, I set up a **Central Store** on my Domain Controller (**`NYCE-DC01`**). This forces Group Policy to pull its template definitions from a single, shared folder in `SYSVOL` instead of local storage.

<div class="callout callout-note"><strong>NOTE:</strong><p style="margin-top: 10px; margin-bottom: 0;">If you want to manage features specific to newer Windows builds (like Windows 11 23H2/24H2), download the latest <strong>Administrative Templates (.admx)</strong> installer directly from Microsoft and extract them into this folder.</p>
</div>

**Any updates made to `.admx` files in `SYSVOL` will automatically apply domain-wide for anyone editing GPOs!**

<iframe width="100%" height="450" src="https://www.youtube.com/embed/BtuXd0MRybI?si=ZgJ2DW5wyfyw0-Gm" title="ADMX Central Store" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 2: Deploying Workstation Baseline Security Policies**
Once the Central Store was up and running, it was time to establish a baseline security policy for my workstations. But before diving into the Group Policy editor, I had to get a clear handle on how GPO settings are structured and where they belong.

* **Getting the Structure Right**
   * <span style="color: #4A90E2;"><strong>Computer Configuration vs. User Configuration</strong></span>
      * **Computer Configuration:** Applies directly to the ***machine itself***, regardless of who signs in. It processes at system startup (e.g., security hardening, firewall rules, local account controls).

      * **User Configuration:** Applies to the ***user account***, following that person no matter which domain machine they log into (e.g., desktop wallpaper, drive mappings, browser settings).

   * <span style="color: #4A90E2;"><strong>Policies vs. Preferences</strong></span>
      * **Policies (Enforced Rules):** Think of this as a strict workplace rule, like ***wearing a mandatory security badge***. It is non-negotiable, locked down, and employees cannot change or turn it off. 

      * **Preferences (Flexible Defaults):** Think of this as the company handing a new employee a desk setup on Day 1. ***This is the initial setup***. They set up your monitor height and give you a default penholder for convenience, but if you want to move the penholder to the left side of your desk, you're free to do so.

<div class="callout callout-danger">
<strong>WARNING:</strong>
<p style="margin-top: 10px; margin-bottom: 0;">
<strong>Do not modify the Default Domain Policy.</strong>
This GPO is linked directly to the root of the domain, meaning every single user and computer processes it. Because of that, it should only be used for four specific areas:<strong><em>account policy settings, password policy, account lockout policy, and Kerberos policy</em></strong>. 
Any other setting—like software installs, desktop configurations, or firewall rules—should go into a separate, focused GPO. When you stuff too many settings into the Default Domain Policy, troubleshooting becomes a nightmare, and it can slow down login performance since every single device and account in the domain is forced to process it. Keeping GPOs small and modular makes management so much easier.</p>
</div>

With that in mind, here are the only **baseline authentication rules** I configured inside the Default Domain Policy:

| Policy Setting | Configuration | Purpose / Enterprise Context |
| :--- |  :--- | :--- |
| **Account Lockout Threshold** | **5 invalid attempts** | Aligns with CIS Benchmarks to prevent brute-force attacks while mitigating Denial of Service (DoS) risk. |
| **Account Lockout Duration** | **15 minutes** | Implements an automatic cooldown period without requiring manual IT Helpdesk intervention. |
| **Reset Lockout Counter After** | **10 minutes** | Sets the observation window before resetting the failed attempts counter back to zero. |
| **Minimum Password Length** | **12 characters** | Replaces legacy 8-character limits to defend against modern GPU-based password hashing tools. |
| **Password Complexity** | **Enabled** | Enforces 3 of 4 character classes (uppercase, lowercase, numbers, special characters). |
| **Enforce Password History** | **24 passwords** | Prevents users from immediately cycling back to previous passwords. |

---

#### 2. Workstation Baseline Controls (Configured in `C_WS_Baseline_Security`)

For machine-specific hardening, I configured local policy rules inside `C_WS_Baseline_Security` to enforce physical and operational workstation security:

| Policy Setting | Path | Configuration | Purpose |
| :--- | :--- | :--- | :--- |
| **Interactive Logon Banner Title** | `Computer Config > Policies > Windows Settings > Security Settings > Local Policies > Security Options` | **"UNAUTHORIZED ACCESS PROHIBITED"** | Heading displayed prior to the Windows logon prompt. |
| **Interactive Logon Banner Text** | `Local Policies > Security Options` | **"This system is restricted to authorized NYCE Home Lab users..."** | Mandatory legal notice required for audit compliance. |
| **Hide Last Signed-In User** | `Local Policies > Security Options` | **Enabled** | Clears the previous user's account name from the login screen to prevent shoulder surfing. |
| **Disable Built-in Guest Account** | `Local Policies > Security Options` | **Disabled** | Closes an unauthenticated local entry point across all domain workstations. |
| **Screen Saver Lock Timeout** | `User Config > Policies > Administrative Templates > Control Panel > Personalization` | **600 seconds (10 mins)** | Automatically locks unattended workstations to safeguard active user sessions. |
| **Logon/Logoff Event Auditing** | `Security Settings > Advanced Audit Policy > Audit Policies > Logon/Logoff` | **Audit Success & Failure** | Generates Event ID 4624/4625 logs critical for SOC/SIEM monitoring and detection. |

---

#### Optimization & OU Linking

1. **Disable Unused Node:** Since `C_WS_Baseline_Security` primarily focuses on machine-level configurations, I right-clicked the GPO in GPMC, navigated to **GPO Status**, and set **User Configuration Settings Disabled** (excluding the global screen lock setting if enforced per computer).  
   > *Why?* Disabling unused nodes speeds up group policy processing time during client startups.

2. **Linking to Target OU:**  
   - In GPMC, right-click the **Workstations** OU $\rightarrow$ Select **Link an Existing GPO...** $\rightarrow$ Choose `C_WS_Baseline_Security`.

---

#### Key Takeaway

> **Architecture Principle:** Never clutter the **Default Domain Policy** with drive mappings, wallpapers, or application policies. Keep it clean for core identity baselines, and scope device hardening to dedicated `C_` and `U_` GPOs linked to their respective Organizational Units.
**What's Next?**  
*Up Next: In **Phase 4: Group Policy & Advanced Security (GPO)**, we will automate configurations, map these shared network drives automatically for our users upon login, and enforce security baselines across the domain workstations.*
