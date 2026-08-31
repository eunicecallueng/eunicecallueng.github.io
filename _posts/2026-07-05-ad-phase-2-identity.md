---
title: "Phase 2: Core Identity & Access Control (ADUC)"
date: 2026-07-09
categories: [Homelab, Active Directory]
tags: [aduc, rbac, user-management, agdlp]
hidden: true
sitemap: false
permalink: /posts/ad-phase-2-identity/
---

In a real-world enterprise environment, you don’t just dump every user and computer into a single default container. You want to organize them logically to make administrative control and security policies way easier to manage down the road.

Once my Domain Controller (**`NYCE-DC01`**) was healthy and resolving DNS correctly, Phase 2 was all about populating the environment. This phase covers onboarding my first workstation, organizing domain resources, provisioning users, and applying least-privilege security controls.


## **Step 1: Organizational Unit (OU) Structure & User Provisioning**
Right after setting up the file server, my next big focus was building out the Organizational Units (OUs) and provisioning users. To mirror standard directory design best practices, I structured my domain (**`nycehomelab.local`**) with a dedicated organizational layout:

* Created a clean, structured Organizational Unit (OU) layout separating regional sites, departments, workstations, and server objects (e.g., **OU=PH ➔ OU=Users, OU=Workstations, OU=Servers**). This makes applying targeted policies so much easier down the road!

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/eDeDZO7Inn0?si=ILkNnxiJN4K2iebM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 2: Implementing the AGDLP Security Model**
Setting up access control was easily one of the most satisfying parts of Phase 2! After organizing my OUs and provisioning users, I established the enterprise-standard **AGDLP** **A**ccount $\rightarrow$ **G**lobal $\rightarrow$ **D**omain Local $\rightarrow$ **P**ermission framework.

* **Why I Chose the AGDLP Strategy**
    * **Role-Based Access Control (RBAC):** Users belong to Global Groups by job role—department transfers only require updating group memberships, not server permissions.
    * **Granular Privilege Management:** Domain Local Groups define exact resource access, keeping permissions explicit and easy to audit.
    * **Effortless Auditing & Onboarding:** When a new team member joins, they automatically inherit all necessary access simply by being added to their role’s global group.

    <div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
    <strong style="font-size: 1.1em;">💡 Rule of Thumb: </strong>
    <p style="margin-top: 10px; margin-bottom: 0;"><strong>Never Grant Direct User Rights:</strong> Always assign users to Global Groups, Global Groups to Domain Local Groups, and Domain Local Groups to folder permissions—never mix these layers.</p>
    </div>

* **How the AGDLP Chain Works in My Lab**
    * **Account (A):** Individual user accounts (e.g., john.doe).
    * **Global Group (G):** Accounts are nested into Global Groups based on job function (e.g., GG-HD-Tech).
    * **Domain Local Group (DL):** Global Groups are nested into Domain Local Groups created for specific resource access (e.g., SH-HR-RW).
    * **Permissions (P):** Explicit NTFS and SMB permissions are assigned directly to the Domain Local Group on the file system folder.

---
---

* **Naming Convention for Group Management:** 
    * I adopted a clean **[Type]-[Scope]-[Resource]-[Access]-[Env]** naming template structure (e.g., SEC-HR-Share-RW-PRD). Keeping the tokens in the exact same order makes searching and auditing in Active Directory a breeze:
        * **Type:** what kind of group (`GG`, `DL`, `LIC`, `ADM`, `APP`, `SH`, `GPO`, `SYNC`)
        * **Scope:** broad boundary or business unit (`GLOBAL`, `EMEA`, `APAC`, `FIN`, `HR`, `IT`, `Sales`)
        * **Resource:** the thing being controlled (`Share`, `App`, `Site`, `DB`, `OU`, `System`)
        * **Access:** permission level (`RO`, `RW`, `OWNER`, `ADMIN`, `CONTRIBUTOR`)
        * **Env/Region:** prod/nonprod or datacenter code (`PRD`, `DEV`, `UAT`)
        * **Qualifier:** optional clarifier (`External`, `Vendor`, `Temporary`)

* **Prefixes and what they mean:**
    *  `GG` - Security group used for access control.
    * `DL`- Distribution (mail) group.
    * `LIC` - Licensing / entitlement group (M365 group-based licensing).
    * `ADM` - Administrative / privileged group.
    * `GPO` - Used to scope or filter Group Policy.
    * `APP` - Application role groups (often nested into access groups).
    * `SH` - File share access groups (if you want a resource-specific type).
    * `SYNC` - Provisioning, sync, or staging groups.

* **Strict Access Level Tokens:** To avoid the usual confusion between terms like "Modify", "Write", or "Full", I locked down permission labels to a clear, standardized set:
    * `RO` - (Read-Only)
    * `RW` - (Read/Write)
    * `OWNER` - manage the resource (often includes RW + ownership tasks)
    * `ADMIN` - administrative control (high privilege, should be rare)

* Enforcing these standards early made nesting groups intuitive and kept the entire security model super clean, predictable, and ready for expansion!

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/cCt0lx_goTg?si=w4KAXxP-ldc1wB7s" title="Implementing the AGDLP Security Model" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 3: Applying Share & NTFS Permissions via AGDLP**
With my AGDLP groups ready, it was time to put them to work on **`NYCE-FS01`**. This is where the real value of the AGDLP model clicked for me!
    
* **Clean Permission Hierarchy:** Kept share-level permissions open (`Full Control` to `Authenticated Users`) and let NTFS handle actual security—preventing rule conflicts.
* **Tighter Security by Default:** Disabled folder inheritance on departmental shares to instantly block unauthorized domain-wide access.
* **Effortless Scalability:** Bound permissions to resource groups (`SH-HD-Team_Data-RO`) instead of users, so access updates happen entirely within Active Directory.
* **Instant, Reliable Protection:** Verified client-side access right away—read-only users were immediately blocked from creating or modifying files.

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/ABnBZa-3_eU?si=Oh3VFTRJGKbWB928" title="Applying Share & NTFS Permissions via AGDLP" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 4: Account Templates for Standardized User Onboarding**
Setting up user accounts manually one by one is time-consuming and prone to human error. To streamline onboarding in my lab, I configured User Account Templates to automate departmental defaults!

* **Why Use User Templates?**
    * **Faster Onboarding:** Creating new accounts takes seconds—just right-click the template and choose Copy to pre-fill standard attributes.
    * **Consistent Security Baseline:** Ensures uniform logon hours, group memberships, and department details across all team members automatically.
    * **Automated Home Directory Creation:** Using %username% in the template’s mapped drive path automatically generates personalized, secured network folders upon account creation.

    <div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
    <strong style="font-size: 1.1em;">💡 Pro Tip: </strong>
    p style="margin-top: 10px; margin-bottom: 0;">Always keep template accounts <strong>disabled</strong> so no one can log in as the template itself, and use a prefix like <em><strong>_Template or tpl_</strong></em> so they sit neatly at the top of your OUs.</p>
    </div>

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/XV69mPq2nw4?si=JZd_fM-L-neQ7fuK" title="Standardizing User Onboarding with Account Templates" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
---
* **Testing Account Templates**
    * To make sure my account templates, logon restrictions, and folder redirection rules were working as intended, I put `maricel.soriano`'s account through a complete end-to-end verification flow from the Windows 11 client VM.
    Here’s a breakdown of the tests I ran and how they played out:
        
        | Test Case | Scenario / Configuration | Expected Result | Actual Result |
        | :--- | :--- | :--- | :--- |
        | **1. Disabled Account Test** | Set account status to **Disabled** in Active Directory. | Block domain logon attempt. | **Passed** *(Received "Your account has been disabled")* |
        | **2. Mandatory Password Change** | Re-enabled account + checked **"Must change password at next logon"**. | Prompt user to set a new password on login. | **Passed** *(Successfully updated password)* |
        | **3. Logon Hours Restriction** | Configured restricted logon hours on the DC. | Block logon outside the allowed schedule. | **Passed** *(Received "Account has time restrictions")* |
        | **4. Folder Redirection & Share Access** | Adjusted logon hours to the current time and logged in. | Auto-map network drive and create a test file. | **Passed** *(File created and synced to `NYCE-FS01`)* |
        
    <iframe width="100%" height="450" src="https://www.youtube.com/embed/dWK9rlK-6mw?si=iAeTo7QHtUEyCMfy" title="User Template Validation and Testing" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 5: Delegating Administrative Control**
Giving domain admin rights to everyone who resets passwords is a huge security risk. To follow the **Principle of Least Privilege**, I used Active Directory’s **Delegation of Control Wizard** to assign specific, scoped administrative tasks directly to our Helpdesk staff.

* **Why Delegate Control?**
    * **Enhanced Security:** Keeps Domain Admin credentials restricted while allowing Tier 1/2 Helpdesk to handle routine administrative requests.
    * **Granular Access**: Limits permissions strictly to specific Organizational Units (OUs) instead of granting domain-wide access.
    * **Operational Efficiency**: Empowers support staff to instantly resolve user account issues without escalating to senior system administrators.

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/-OiREKXbp6U?si=qmYPqG86dRw26jGa" title="Delegating Administrative Control" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

* **RSAT Installation**
    * To test and validate if our Delegation of Control settings are actually working, we first need to install RSAT (Remote Server Administration Tools) on our client VM. 
        * Setting Up RSAT on the Client VM
            * You can normally install RSAT via **Windows Settings > Optional features**. However, to keep things lightweight, I initially tried installing only the specific Active Directory RSAT capability via PowerShell. 
            * I ran into a bit of an issue where individual feature targeting wasn't processing properly in my environment. To work around this, I executed a broader query to fetch and install the full Active Directory RSAT package:

            ```powershell
            Get-WindowsCapability -Name RSAT.ActiveDirectory* -Online | Add-WindowsCapability -Online
            ```

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/Wkw2CWJ2muw?si=tI3XfzVH_C0-XaFv" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

--- 

* **Testing & Validating Delegation of Control**
    * With RSAT ready, I tested our delegated Helpdesk account (NYCEHOMELAB\john.doe) to verify that least privilege enforcement was working as intended:

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/8Om3uOM5TA0?si=5nmipNJvWtUtcxkv" title="Delegated Control Testing" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

**What's Next?**  
*Up Next: In **Phase 3: Storage & File Sharing**, we will put these newly created Security Groups to work! I'll walk through configuring centralized file servers and enforcing strict role-based access controls across our shared network drives.*