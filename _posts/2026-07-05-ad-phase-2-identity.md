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

## **Step 1: Preparing the File Server & Storage Infrastructure**
My very first step was preparing the file server architecture. One big lesson I've learned in tech is that designing your server infrastructure properly from day one saves you from massive headaches later—especially when it comes to security, stability, and scalability.

1. **Why I Separated the Domain Controller and File Server Roles**

    Technically, a single Windows Server can host both Active Directory (AD DS) and File Services. But for my lab, I chose to separate them across dedicated virtual machines (VMs) because it mirrors actual enterprise best practices:
    
    * **Ransomware & Malware Containment:** File servers are usually the main target for user-driven malware. By keeping my File Server on a separate VM, an infection can't easily reach or encrypt my Active Directory database (`NTDS.dit`).
    * **Privilege Escalation Prevention:** Domain Controllers don't have a local Administrators group. If you start handing out admin rights for file sharing on a DC, you inadvertently end up granting elevated Domain Admin privileges.
    * **Reduced Attack Surface**: File servers need open SMB ports for workstations to connect. Separating this role protects my DC from unauthorized network scans and lateral movement.
    * **Performance & Resource Allocation:** **`NYCE-DC01`** gets dedicated CPU and RAM for fast authentication and DNS queries, without getting bogged down by heavy disk I/O when users transfer large files.

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/RR-WV6euKrA?si=ynbOlv1s5v0cz9Gb" title="File Server Creation" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

2. **Isolating Shared Folders to a Dedicated Data Volume**

    > Another rule of thumb I followed: never dump shared folders or user data onto the system volume (C: Drive). I allocated a dedicated data volume (E: drive) for all my network shares instead:

    * **Operating System Protection:** If users ever fill up the shared storage space, the system volume stays completely untouched, preventing OS crashes and core service failures.
    * **Simplified Security & Quotas:** Setting up NTFS permissions, Disk Quotas, and File Screening rules on a standalone data volume keeps things clean and prevents accidental tweaks to system files (C:\Windows).
    * **Storage Performance & Maintenance:** A dedicated disk volume gives me dedicated disk I/O throughput for SMB traffic. Plus, it makes backing up, expanding, or restoring storage way smoother if something goes wrong.

    
* **Here's How I Added a Virtual Hard Drive in VMware:**
    1. Shut down the File Server.
    2. Right-click the `NYCE-FS01` VM and select **Settings**.
    3. Under the **Hardware** tab, click **Add**
    4. Choose **Hard Disk** $\rightarrow$ **Next**
    5. Select **SCSI** (Recommended) $\rightarrow$ **Next**.
    6. Select **Create a new virtual disk** $\rightarrow$ **Next**.
    7. Set the **disk size** (e.g., 15 GB) and select **Store virtual disk as a single file**.
    8. Confirm the file path, click **Finish**, and select **OK** to apply VM settings.
    9. **You can follow the rest of the step-by-step walkthrough in the video below:**

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/K4j5J5vrSzE?si=QEHCdvwjfxDoAjvf" title="Isolating Shared Folders to a Dedicated Data Volume" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---


## **Step 2: Organizational Unit (OU) Structure & User Provisioning**
Right after setting up the file server, my next big focus was building out the Organizational Units (OUs) and provisioning users. To mirror standard directory design best practices, I structured my domain (**`nycehomelab.local`**) with a dedicated organizational layout:

* Created a clean, structured Organizational Unit (OU) layout separating regional sites, departments, workstations, and server objects (e.g., **OU=PH ➔ OU=Users, OU=Workstations, OU=Servers**). This makes applying targeted policies so much easier down the road!

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/eDeDZO7Inn0?si=ILkNnxiJN4K2iebM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 3: Implementing the AGDLP Security Model**
Setting up access control was easily one of the most satisfying parts of Phase 2! After organizing my OUs and provisioning users, the next critical step was establishing a proper authorization model.

Instead of assigning access rights directly to individual user accounts—which quickly turns into an administrative nightmare—I implemented the enterprise-standard **AGDLP** (**A**ccount $\rightarrow$ **G**lobal $\rightarrow$ **D**omain Local $\rightarrow$ **P**ermission) framework.

* **Why I Chose the AGDLP Strategy**
    * In a real-world enterprise network, assigning permissions directly to end users leads to permission bloat, orphaned access rights, and massive headaches during audits. AGDLP keeps access management clean, scalable, and easy to troubleshoot:
        * **Role-Based Access Control (RBAC):** Users are assigned to Global Groups based on their job roles (e.g., HR, Finance, IT). If an employee moves to a new department, I simply update their group membership—no need to touch folder permissions on the server.
        * **Granular Privilege Management:** Domain Local Groups define the actual permissions on resources (like read/write access to a specific network share). This clear separation ensures that access rights are explicit and easy to trace.
        * **Effortless Auditing & Onboarding:** When a new team member joins, they automatically inherit all necessary access simply by being added to their role’s global group.
* **How the AGDLP Chain Works in My Lab**
    * **Account (A):** Individual user accounts (e.g., john.doe).
    * **Global Group (G):** Accounts are nested into Global Groups based on job function (e.g., GG-HD-Tech).
    * **Domain Local Group (DL):** Global Groups are nested into Domain Local Groups created for specific resource access (e.g., SH-HR-RW).
    * **Permissions (P):** Explicit NTFS and SMB permissions are assigned directly to the Domain Local Group on the file system folder.


* **Predictable Naming Template:** I adopted a clean **[Type]-[Scope]-[Resource]-[Access]-[Env]** naming template structure (e.g., SEC-HR-Share-RW-PRD). Keeping the tokens in the exact same order makes searching and auditing in Active Directory a breeze:
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

## **Step 4: Applying Share & NTFS Permissions via AGDLP**



## **Step 2: Streamlining User Provisioning**
Created standardized user templates (_Template) per department to make onboarding seamless, alongside active test user accounts (like john.d for HR) assigned to their respective GG- groups


## **Step 2: Domain Join & Client Network Configuration**

Building on the foundation from Phase 1, I brought my client machine (my Windows 11 VM) into the newly minted environment. Before any machine can talk to a domain controller, getting the network and DNS pointing correctly is crucial:

* **Configuring Client DNS:**
    * I opened network properties on the Windows 11 client and pointed its Preferred DNS Server directly to my Domain Controller’s static IP address (``192.168.120.100``). This ensures the client can properly query and resolve the Active Directory domain.

        <iframe width="100%" height="450" src="https://www.youtube.com/embed/wu-0Izl8QDU?si=7WFvcqeab0K3i2NF" title="Client VM DNS coniguration" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

* **Joining and Logging In:** 
    * After joining the machine to ``nycehomelab``.local, I tested the environment by logging into the client VM using a custom domain user account (``john.doe``) that I set up in Active Directory, verifying that centralized authentication was working seamlessly.

        <iframe width="100%" height="450" src="https://www.youtube.com/embed/lsE92TkJu_Y?si=BG69DJW2sgD6Rymm" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 3: Role-Based Access Control (RBAC) & Delegation**
To implement the principle of least privilege, permissions were assigned using security groups rather than individual account rights, and specific admin duties were delegated down.:

* **Security Group Management:** 
    * I created security groups for departmental access and admin roles using the AGDLP strategy to keep everything clean and scalable. Since AGDLP is such a fun and important concept in AD, I broke down how I set it up on a separate AGDLP Deep-Dive Page!

* **Delegating Control:** 
    * I used the Delegation of Control Wizard in ADUC to give our Helpdesk team permission to reset passwords for specific OUs. It was awesome seeing this in action because it meant giving them the exact rights they needed without handing over full Domain Admin privileges.




---
**What's Next?**  
*Up Next: In **Phase 3: Storage & File Sharing**, we will put these newly created Security Groups to work! I'll walk through configuring centralized file servers and enforcing strict role-based access controls across our shared network drives.*