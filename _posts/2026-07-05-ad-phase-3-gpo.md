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

* **How to Set Up the Central Store**

1. **Locate the Local Policy Definitions:**  
   On `NYCE-DC01`, navigate to the local template repository:  
   `C:\Windows\PolicyDefinitions`

2. **Create the Central Store Directory in SYSVOL:**  
   Open File Explorer and navigate to the domain policies directory:  
   `\\NYCE-DC01\SYSVOL\nycehomelab.local\policies`  
   Inside this folder, create a new folder named `PolicyDefinitions`.

3. **Copy the Administrative Templates:**  
   Copy all contents (including all `.admx` files and language folders like `en-US`) from `C:\Windows\PolicyDefinitions` into the newly created `PolicyDefinitions` folder in `SYSVOL`.

   > **Note:** If you want to manage features specific to newer Windows builds (like Windows 11 23H2/24H2), download the latest **Administrative Templates (.admx)** installer directly from Microsoft and extract them into this folder.

#### Verification

To confirm that Active Directory is now fetching template definitions from the network share instead of the local machine:

1. Open **Group Policy Management** (`gpmc.msc`) on `NYCE-DC01`.
2. Right-click any existing GPO (or create a temporary test GPO) and click **Edit**.
3. Expand **Computer Configuration > Policies > Administrative Templates**.
4. Check the top node description. It should explicitly state:  
   `Policy definitions (ADMX files) retrieved from the central store.`

Once verified, any updates made to `.admx` files in `SYSVOL` will automatically apply domain-wide for anyone editing GPOs!



---

**What's Next?**  
*Up Next: In **Phase 4: Group Policy & Advanced Security (GPO)**, we will automate configurations, map these shared network drives automatically for our users upon login, and enforce security baselines across the domain workstations.*
