---
title: "Phase 1: Foundation & Domain Join"
date: 2026-07-05
categories: [Homelab, Active Directory]
tags: [windows-server, vmware, domain-controller]
hidden: true
sitemap: false
permalink: /posts/ad-phase-1-foundation/
---

##    When I first decided to dive into Active Directory, I quickly learned that theory is great, but getting your hands dirty in a homelab is where the real learning happens. 
## In this first phase, we lay down the core infrastructure needed before anything else can work.
## Here is a breakdown of what I did and the crucial pre-requisites required to get a domain controller up and running smoothly:

# **Step 1: Server Provisioning & Pre-Requisites**

<div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
  <strong style="font-size: 1.1em;">💡 Best Practice Tip:</strong>
  <p style="margin-top: 10px; margin-bottom: 0;">Best practice requires assigning a proper hostname and a static IP address <strong>before promoting the server to a Domain Controller.</strong></p>
</div>

## I started by updating the default computer name in Server Manager (Local Server) to something recognizable: **`NYCE-DC01`**

   **Change Hostname:**
   * Open **Server Manager** > **Local Server**.
   * Click the default Computer Name (e.g., `WIN-XXXXXXX`).
   * Rename the computer to your preference, I renamed it to **`NYCE-DC01`** and **restart the virtual machine**.

      ![Rename the server](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Rename-the-Server.jpg)

## Active Directory relies heavily on stable, predictable network addressing so I assigned a **`static IP`** address and pointed the Preferred DNS back to **`localhost (127.0.0.1)`** since this server will double as our DNS resolver.
   
   **Configure Static IP Address:**
   * Press `Win + R`, type `ncpa.cpl`, and press **Enter**.
   * Right-click your network adapter > **Properties** > **Internet Protocol Version 4 (TCP/IPv4)**.
   * Manually assign a static IP address (e.g., IP: `192.168.120.100`, Subnet: `255.255.255.0`, Gateway: `192.168.120.1`, Preferred DNS: `127.0.0.1`).

      ![Configure-Static-IP](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Configure-Static-IP.jpg)

---

# **Step 2: Active Directory Domain Services (AD DS) Installation**

## Inside the Server Manager, I went through Add Roles and Features, checked Active Directory Domain Services, and let the wizard install the necessary binaries and management tools
1. In **Server Manager**, click **Manage** > **Add Roles and Features**.

   ![Add Features](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Add-Features.jpg)
   
2. Progress through the wizard until reaching **Server Roles**.
3. Check **Active Directory Domain Services** (click **Add Features** on the pop-up).
4. Click **Install** and wait for completion.

## Once installed, I triggered the yellow notification flag to promote the server to a domain controller
1. Click the **Yellow Notification Flag** in Server Manager.
2. Select **Promote this server to a domain controller**.

   ![Promote to DC](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Promote-to-Domain-Controller.png)
   
3. Choose **Add a new forest** and enter your domain name:
   ##   **Root Domain Name:** `nycehomelab.local`
4. Set a strong **DSRM (Directory Services Restore Mode) Password**.
5. Leave remaining options at default and click **Install**. The server will automatically reboot.

---

# **Step 3: DNS Configuration & Name Resolution**

## Since DNS is the backbone of Active Directory, I made sure to verify that the DNS service was running and integrated cleanly with the new forest so my network machines could find service locations:

1. Log into the server using domain administrator credentials: `NYCEHOMELAB\Administrator`.

   * **Check DNS Manager:** I opened DNS Manager (via ``Server Manager > Tools > DNS``), expanded Forward Lookup Zones, and opened nycehomelab.local to verify critical AD service folders like ``_msdcs``, ``_sites``, ``_tcp``, and ``_udp`` were right where they needed to be.

   * **Test Name Resolution:** I fired up Command Prompt and ran an ``nslookup`` for my domain (``nycehomelab.local``) to make sure it resolved straight to my DC's IP address..

   * Verify Service Status: I quickly checked ``services.msc`` to confirm that the DNS Server service was *running* and set to *Automatic*.

   <video controls width="100%">
   <source src="/assets/media/vid/server-dns-configuration.mp4" type="video/mp4">
   </video>

---

# **Step 4: 4. Domain Join & Client Network Configuration**

## To wrap up Phase 1, I brought my client machine (my Windows 11 VM) into the newly minted environment. Before any machine can talk to a domain controller, getting the network and DNS pointing right is everything:

   * **Configuring Client DNS:** I opened network properties on the Windows 11 client and pointed its Preferred DNS Server directly to my Domain Controller’s static IP address (``192.168.120.100``). This ensures the client can properly query and resolve the Active Directory domain.

      <iframe width="100%" height="450" src="https://www.youtube.com/embed/wu-0Izl8QDU?si=7WFvcqeab0K3i2NF" title="Client VM DNS coniguration" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

   * **Joining and Logging In:** After joining the machine to ``nycehomelab``.local, I tested the environment by logging into the client VM using a custom domain user account (``john.doe``) that I set up in Active Directory, verifying that centralized authentication was working seamlessly.

      <iframe width="100%" height="450" src="https://www.youtube.com/embed/lsE92TkJu_Y?si=BG69DJW2sgD6Rymm" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---
[← Back to AD Series Overview](/posts/active-directory-series/)
