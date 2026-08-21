---
title: "Phase 1: Foundation & Domain Join"
date: 2026-07-05
description: >-
   When I first decided to dive into Active Directory, I quickly learned that theory is great, but getting your hands dirty in a homelab is where the real learning happens.<br\>
   
   In this first phase, we lay down the core infrastructure needed before anything else can work.
categories: [Homelab, Active Directory]
tags: [windows-server, vmware, domain-controller]
hidden: true
sitemap: false
permalink: /posts/ad-phase-1-foundation/
---

[← Back to AD Series Overview](/posts/active-directory-series/)

---

## Here is a breakdown of what I did and the crucial pre-requisites required to get a domain controller up and running smoothly:

# **Step 1: Preparing the Server (Pre-Requisites)**

> [!IMPORTANT]
> <div style="background-color: #99cc33; color: #000000; padding: 20px 12px; border-radius: 6px; font-weight: 500;">Best practice requires assigning a proper hostname and a static IP address <em>before promoting the server to a Domain Controller.</em></div>

## I started by updating the default computer name in Server Manager (Local Server) to something recognizable: `NYCE-DC01`

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

# **Step 2: Installing & Promoting AD DS**

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

# **Step 3: Testing & Verification**

1. Log into the server using domain administrator credentials: `NYCEHOMELAB\Administrator`.
2. Open **Server Manager** > **Tools**.
3. Verify the following tools are present and functional:
   - [x] Active Directory Users and Computers
   - [x] DNS Manager
   - [x] Group Policy Management


