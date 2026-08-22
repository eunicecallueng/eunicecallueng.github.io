---
title: "Phase 3: Storage & File Sharing"
date: 2026-07-11
categories: [Homelab, Active Directory]
tags: [file-server, ntfs, share-permissions, rbac]
hidden: true
sitemap: false
permalink: /posts/ad-phase-3-storage/
---

<div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
  <strong style="font-size: 1.1em;">💡 Best Practice Tip: Principle of Least Privilege (PoLP)</strong>
  <p style="margin-top: 10px; margin-bottom: 0;">When securing resources, it's best to configure broad access (like setting <em><strong>Everyone: Full Control</strong></em>) strictly at the <strong>Share Permissions</strong> level. You then restrict the actual granular access using <strong>NTFS Security ACLs</strong> tied to your AD Security Groups. This prevents conflicting permissions and keeps access clean!</p>
</div>

Now that our users and security groups are neatly organized in Active Directory (Phase 2), it's time to give them something to access securely. 

In this phase, I configured a centralized File Server role and applied Role-Based Access Control (RBAC) to ensure users only have access to what they need for their jobs.

### **Step 1: Provisioning the Shared Folders**

I started by creating dedicated directories on the server for different departments. For this lab environment, I created a specific share (e.g., `HR_Share` or `IT_Share`) on the server's data drive to act as our centralized file repository.

### **Step 2: Configuring Share Permissions**

To make the folder accessible over the network, I first configured the broad Share Permissions:

* Right-clicked the folder > **Properties** > **Sharing** > **Advanced Sharing**.
* Checked **Share this folder**.
* Under **Permissions**, I set **Everyone** to **Full Control** (or Change). 

*(Note: While giving "Everyone" access sounds risky, network share permissions simply act as a front door. The actual strict security is locked down in the next step using NTFS permissions).*

### **Step 3: Hardening Access with NTFS Security Permissions**

This is where the Principle of Least Privilege (PoLP) comes into play. I navigated to the **Security** tab of the folder properties to restrict access exclusively to the Security Groups I created in Phase 2.

* **Disabled Inheritance:** I clicked **Advanced** > **Disable inheritance** to stop default permissions from trickling down from the parent drive.
* **Removed General Access:** I removed default broad groups like `Users` to prevent unauthorized access.
* **Assigned Group Permissions:** I added the specific AD Security Group (e.g., `SG-IT-Department`) and granted them **Modify** access. Now, only members of this designated group can read, write, or delete files in this folder.

*(Insert your Step 3 Video Walkthrough here)*

### **Step 4: Verifying Client Access**

To prove the permissions work in a real-world scenario, I jumped back into my Windows 11 client machine:

* Logged in as the domain test user (`john.doe`).
* Accessed the network share path (`\\192.168.120.100\HR_Share` or `\\NYCE-DC01\HR_Share`) via the Run dialog (`Win + R`).
* Verified that I could create and modify files if the user was in the correct group. I also tested an out-of-group account to ensure it successfully threw an "Access Denied" prompt, proving the security works.

*(Insert your Step 4 Video Walkthrough here)*

<div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
  <strong style="font-size: 1.1em;">💡 Homelab Takeaway:</strong>
  <p style="margin-top: 10px; margin-bottom: 0;">Separating <strong>Share Permissions</strong> and <strong>NTFS Permissions</strong> is a fundamental Windows Server concept. By doing this hands-on, the difference between simply "sharing a folder on the network" and applying "strict file-level security" becomes incredibly clear.</p>
</div>

---

**What's Next?**  
*Up Next: In **Phase 4: Group Policy & Advanced Security (GPO)**, we will automate configurations, map these shared network drives automatically for our users upon login, and enforce security baselines across the domain workstations.*



[← Back to AD Series Overview](/posts/active-directory-series/)