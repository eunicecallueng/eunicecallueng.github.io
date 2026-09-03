---
title: "Phase 1: Server Provisioning, Storage & Client Onboarding"
date: 2026-07-07
categories: [Homelab, Active Directory]
tags: [windows-server, vmware, domain-controller]
hidden: true
sitemap: false
permalink: /posts/ad-phase-1-foundation/
---

When I first decided to dive into Active Directory, I quickly learned that theory is great, but getting your hands dirty in a homelab is where the real learning happens. 

In this first phase, we lay down the core infrastructure needed before anything else can work.

Here is a breakdown of what I did and the crucial pre-requisites required to get a domain controller up and running smoothly:

## **Step 1: Server Provisioning & Pre-Requisites**

<div class="callout callout-important">
<strong>💡 Best Practice Tip:</strong>
<p style="margin-top: 10px; margin-bottom: 0;">Best practice requires assigning a proper hostname and a static IP address <strong>before promoting the server to a Domain Controller.</strong></p>
</div>

I started by updating the default computer name in Server Manager (Local Server) to something recognizable: **`NYCE-DC01`**

   **Change Hostname:**
   * Open **Server Manager** > **Local Server**.
   * Click the default Computer Name (e.g., `WIN-XXXXXXX`).
   * Rename the computer to your preference, I renamed it to **`NYCE-DC01`** and **restart the virtual machine**.

      ![Rename the server](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Rename-the-Server.jpg)

---

Active Directory relies heavily on stable, predictable network addressing so I assigned a **`static IP`** address and pointed the Preferred DNS back to **`localhost (127.0.0.1)`** since this server will double as our DNS resolver.
   
   **Configure Static IP Address:**
   * Press `Win + R`, type `ncpa.cpl`, and press **Enter**.
   * Right-click your network adapter > **Properties** > **Internet Protocol Version 4 (TCP/IPv4)**.
   * Manually assign a static IP address (e.g., IP: `192.168.1.100`, Subnet: `255.255.255.0`, Gateway: `192.168.1.1`, Preferred DNS: `127.0.0.1`).

      ![Configure-Static-IP](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Configure-Static-IP.jpg)

---

## **Step 2: Active Directory Domain Services (AD DS) Installation**

Inside the Server Manager, I went through Add Roles and Features, checked Active Directory Domain Services, and let the wizard install the necessary binaries and management tools
1. In **Server Manager**, click **Manage** > **Add Roles and Features**.

   ![Add Features](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Add-Features.jpg)
   
2. Progress through the wizard until reaching **Server Roles**.
3. Check **Active Directory Domain Services** (click **Add Features** on the pop-up).
4. Click **Install** and wait for completion.

---

Once installed, I triggered the yellow notification flag to promote the server to a domain controller
1. Click the **Yellow Notification Flag** in Server Manager.
2. Select **Promote this server to a domain controller**.

   ![Promote to DC](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Promote-to-Domain-Controller.png)
   
3. Choose **Add a new forest** and enter your domain name:

         Root Domain Name: nycehomelab.local
4. Set a strong **DSRM (Directory Services Restore Mode) Password**.
5. Leave remaining options at default and click **Install**. The server will automatically reboot.

---

## **Step 3: DNS Configuration & Name Resolution**

Since DNS is the backbone of Active Directory, I made sure to verify that the DNS service was running and integrated cleanly with the new forest so my network machines could find service locations:

Log into the server using domain administrator credentials: `NYCEHOMELAB\Administrator`.

   * **Check DNS Manager:** I opened DNS Manager (via ``Server Manager > Tools > DNS``), expanded Forward Lookup Zones, and opened nycehomelab.local to verify critical AD service folders like ``_msdcs``, ``_sites``, ``_tcp``, and ``_udp`` were right where they needed to be.

   * **Test Name Resolution:** I fired up Command Prompt and ran an ``nslookup`` for my domain (``nycehomelab.local``) to make sure it resolved straight to my DC's IP address..

   * **Verify Service Status:** I quickly checked ``services.msc`` to confirm that the DNS Server service was *running* and set to *Automatic*.

   <video controls width="100%">
   <source src="/assets/media/vid/server-dns-configuration.mp4" type="video/mp4">
   </video>

---

## **Step 4: Preparing the File Server & Storage Infrastructure**
My next step was preparing the file server architecture. One big lesson I've learned in tech is that designing your server infrastructure properly from day one saves you from massive headaches later—especially when it comes to security, stability, and scalability.

<span style="color: #4A90E2;"><strong>1. Why I Separated the Domain Controller and File Server Roles</strong></span>
   * Technically, a single Windows Server can host both Active Directory (AD DS) and File Services. But for my lab, I chose to separate them across dedicated virtual machines (VMs) because it mirrors actual enterprise best practices:
      * **Ransomware & Malware Containment:** File servers are usually the main target for user-driven malware. By keeping my File Server on a separate VM, an infection can't easily reach or encrypt my Active Directory database (`NTDS.dit`).
      * **Privilege Escalation Prevention:** Domain Controllers don't have a local Administrators group. If you start handing out admin rights for file sharing on a DC, you inadvertently end up granting elevated Domain Admin privileges.
      * **Reduced Attack Surface**: File servers need open SMB ports for workstations to connect. Separating this role protects my DC from unauthorized network scans and lateral movement.
      * **Performance & Resource Allocation:** **`NYCE-DC01`** gets dedicated CPU and RAM for fast authentication and DNS queries, without getting bogged down by heavy disk I/O when users transfer large files.

<iframe width="100%" height="450" src="https://www.youtube.com/embed/RR-WV6euKrA?si=ynbOlv1s5v0cz9Gb" title="File Server Creation" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

<span style="color: #4A90E2;"><strong>2. Isolating Shared Folders to a Dedicated Data Volume</strong></span>
   
<div class="callout callout-important">
   <strong>💡 Rule of Thumb:</strong>
   <p style="margin-top: 10px; margin-bottom: 0;"><strong>Never dump shared folders or user data onto the system volume (C: Drive)</strong>. I allocated a dedicated data volume (E: drive) for all my network shares instead:</p>
</div>

   * **Operating System Protection:** If users ever fill up the shared storage space, the system volume stays completely untouched, preventing OS crashes and core service failures.
   * **Simplified Security & Quotas:** Setting up NTFS permissions, Disk Quotas, and File Screening rules on a standalone data volume keeps things clean and prevents accidental tweaks to system files (C:\Windows).
   * **Storage Performance & Maintenance:** A dedicated disk volume gives me dedicated disk I/O throughput for SMB traffic. Plus, it makes backing up, expanding, or restoring storage way smoother if something goes wrong.

    
* **Here's How I Added a Virtual Hard Drive in VMware:**
    1. Shut down the File Server.
    2. Right-click the **`NYCE-FS01`** VM and select **Settings**.
    3. Under the **Hardware** tab, click **Add**
    4. Choose **Hard Disk** $\rightarrow$ **Next**
    5. Select **SCSI** (Recommended) $\rightarrow$ **Next**.
    6. Select **Create a new virtual disk** $\rightarrow$ **Next**.
    7. Set the **disk size** (e.g., 15 GB) and select **Store virtual disk as a single file**.
    8. Confirm the file path, click **Finish**, and select **OK** to apply VM settings.
    9. **You can follow the rest of the step-by-step walkthrough in the video below:**

<iframe width="100%" height="450" src="https://www.youtube.com/embed/K4j5J5vrSzE?si=QEHCdvwjfxDoAjvf" title="Isolating Shared Folders to a Dedicated Data Volume" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---


## **Step 5: Domain Join & Client Network Configuration**
With the server infrastructure ready, it was time to bring my Windows 11 VM into the domain! Before any machine can talk to a domain controller, getting the network and DNS pointing correctly is crucial:

* **Configuring Client DNS:**
    * I opened network properties on the Windows 11 client and pointed its Preferred DNS Server directly to my Domain Controller’s static IP address (``192.168.1.100``). This ensures the client can properly query and resolve the Active Directory domain.

* **Joining and Logging In:** 
    * After confirming that the client VM can ping **`nycehomelab.local`**, it's time to join it to the domain

      <iframe width="100%" height="450" src="https://www.youtube.com/embed/DgHRKRH8iEQ?si=qmfLB-s_I8h5ITYJ" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


[← Back to AD Series Overview](/posts/active-directory-series/)
