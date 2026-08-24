---
title: "Phase 3: Storage & File Sharing"
date: 2026-07-11
categories: [Homelab, Active Directory]
tags: [file-server, ntfs, share-permissions, rbac]
hidden: true
sitemap: false
permalink: /posts/ad-phase-3-storage/
---

Now that our users and security groups are neatly organized in Active Directory (Phase 2), it's time to provide them with secure access to resources. 

In this phase, I configured a centralized file repository and applied Role-Based Access Control (RBAC) to ensure users only access the data they need.


## **Step 1: Adding and Formatting a Dedicated Storage Disk**
Before creating shared folders, it is an enterprise best practice to ***separate user data from the main OS drive***. I provisioned a new virtual hard disk, brought it online via Disk Management in Windows Server, initialized it, and formatted it with an NTFS file system to prepare it for network sharing:

  <iframe width="100%" height="450" src="https://www.youtube.com/embed/_lbv9ivJtmo?si=B9NIRFYAV07etgji" title="Adding and Formatting a Dedicated Storage Disk" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 2: Configuring Share Permissions**

To make the folder accessible over the network, I first configured the broad Share Permissions:

* Right-clicked the folder > **Properties** > **Sharing** > **Advanced Sharing**.
* Checked **Share this folder**.
* Under **Permissions**, I set **Everyone** to **Full Control** (or Change). 

***(Note: While giving "Everyone" access sounds risky, network share permissions simply act as a front door. The actual strict security is locked down in the next step using NTFS permissions).***

  <iframe width="100%" height="450" src="https://www.youtube.com/embed/k7cGiM58sBI?si=60vvQSmfUlALjYCs" title="Configuring Share Permissions" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 3: Hardening Access with NTFS Security Permissions**

This is where the Principle of Least Privilege (PoLP) comes into play. I navigated to the **Security** tab of the folder properties to restrict access exclusively to the Security Groups I created in Phase 2.

* **Disabled Inheritance:** I clicked **Advanced** > **Disable inheritance** to stop default permissions from trickling down from the parent drive.
* **Removed General Access:** I removed default broad groups like `Users` to prevent unauthorized access.
* **Assigned Group Permissions:** I added the specific AD Security Group (e.g., `SG-IT-Department`) and granted them **Modify** access. Now, only members of this designated group can read, write, or delete files in this folder.

<iframe width="100%" height="450" src="https://www.youtube.com/embed/5wskpyCVVRk?si=6kMWFT-tkh_RIjZq" title="Hardening Access with NTFS Security Permissions" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 4: Verifying Client Access**

To prove the permissions work in a real-world scenario, I jumped back into my Windows 11 client machine:

* Logged in as the domain test user (`john.doe`).
* Accessed the network share path (`\\192.168.120.100\HR_Share` or `\\NYCE-DC01\HR_Share`) via the Run dialog (`Win + R`).
* Verified that I could create and modify files if the user was in the correct group. I also tested an out-of-group account to ensure it successfully threw an "Access Denied" prompt, proving the security works.

<iframe width="100%" height="450" src="https://www.youtube.com/embed/KrYaMs_vk7Q?si=zFLJuornzWd7O5D8" title="Verifying Client Access" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
  <strong style="font-size: 1.1em;">💡 Homelab Takeaway:</strong>
  <p style="margin-top: 10px; margin-bottom: 0;">Separating <strong>Share Permissions</strong> and <strong>NTFS Permissions</strong> is a fundamental Windows Server concept. By doing this hands-on, the difference between simply "sharing a folder on the network" and applying "strict file-level security" becomes incredibly clear.</p>
</div>

---

**What's Next?**  
*Up Next: In **Phase 4: Group Policy & Advanced Security (GPO)**, we will automate configurations, map these shared network drives automatically for our users upon login, and enforce security baselines across the domain workstations.*


<div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
  <strong style="font-size: 1.1em;">💡 Best Practice Tip: Principle of Least Privilege (PoLP)</strong>
  <p style="margin-top: 10px; margin-bottom: 0;">When securing resources, it's best to configure broad access (like setting <em><strong>Everyone: Full Control</strong></em>) strictly at the <strong>Share Permissions</strong> level. You then restrict the actual granular access using <strong>NTFS Security ACLs</strong> tied to your AD Security Groups. This prevents conflicting permissions and keeps access clean!</p>
</div>



[← Back to AD Series Overview](/posts/active-directory-series/)