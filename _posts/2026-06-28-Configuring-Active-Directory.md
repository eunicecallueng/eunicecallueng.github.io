---
title: "Setting up Windows Server 2022 in VMware Workstation"
date: 2026-06-28
categories: [Infrastructure, Virtualization]
tags: [vmware, windows-server, active-directory, homelab, networking]
description: >- 
   This is a step-by-step documentation of how I successfully installed Windows Server 2022 inside VMware Workstation Pro and promote it to a Domain Controller.
---
  


## Prerequisites & Free Tools I Used

Before starting with Active Directory, you'll need to have your hypervisor ready and download the Windows Server ISO:

* **Hypervisor:** VMware Workstation Pro  
  *(If you haven't installed it yet, check out my other guide: [VMware Workstation Pro](https://github.com/eunicecallueng/Virtualization-Labs/blob/main/01.VMware-Workstation-Pro/README.md))
* **Operating System:** [Windows Server 2022 ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) (Microsoft gives a free 180-day trial!)


---

## Setting Up the Virtual Machine Container

Here is how I configured the VM settings in VMware before booting up the OS:

1. **Launch VMware Workstation** — Click on **Create a New Virtual Machine** (or hit `Ctrl + N`) to open the setup wizard.
2. **Choose Configuration Type** — Since this is just for my lab experiment, I went with **Typical (recommended)** and clicked **Next**.
3. **OS Installation Selection** — I selected **"I will install the operating system later"**.
 
   ![I will install the operating system later](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/install-os-later.jpg)

   > 💡 *Pro-Tip:* Doing this skips VMware's "Easy Install" feature, which often causes annoying errors during Windows Server installation!
4. **Select Guest OS** — I picked **Microsoft Windows**, and set the version to **Windows Server 2022**, then hit **Next**.
5. **Set VM Name & Location** — Name your VM whatever you like and click **Next**.
6. **Allocate Disk Space** — I set the maximum disk size to **30 GB** (which is plenty for a basic lab environment) and clicked **Next**.
7. **Attach the ISO File** — Before finishing, we need to make sure the VM actually boots from our downloaded Windows Server ISO:
   * Click **Customize Hardware...**
   
     ![Customize Hardware](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Customize-Hardware.jpg)
     
   * Select **New CD/DVD (SATA)** from the left menu.
   * Under **Connection** on the right side, choose **Use ISO image file**.
   * Click **Browse...** and select your downloaded Windows Server 2022 `.iso` file.

     ![ISO Image file](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/ISO-image-file.jpg)
     
8. **Finish Up** — Click **Close**, then hit **Finish**!

---

## Installing & Setting Up Windows Server 2022

Now that our VM container is ready, it's time to fire it up and install the OS:

1. **Wait for the VM to automatically boot up** — Press any key **immediately** when prompted to boot from the CD/DVD.
2. **Basic Setup** — Choose your preferred language, time zone, and keyboard layout, then click **Next** > **Install Now**.

     ![Setting up Win Server](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Setting-up-Win-Server.jpg)
  
3. **Select Edition** — Make sure to select **Windows Server 2022 (Desktop Experience)** so you get the full Graphical User Interface (GUI) instead of just a command prompt!
4. **Installation Type** — Pick **Custom: Install Windows only (advanced)**.
5. **Select Storage Drive** — Double-check that you're picking the **30 GB** drive we allocated earlier in Step 6, then click **Next**.
6. **Wait for Installation** — Grab a coffee ☕ while Windows finishes copying files and reboots.

     ![Installing](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Installing.jpg)

7. **Set Administrator Account** — Upon first startup, create a strong local **Administrator Password**.

And that's it! 🎉 We now have a fresh Windows Server 2022 instance running smoothly in VMware.

---


## Step 2: Preparing the Server (Pre-Requisites)

> [!NOTE]
> <div style="background-color: #5b8500; color: #000000; padding: 10px 12px; border-radius: 6px; font-weight: 500;">Best practice requires assigning a proper hostname and a static IP address <em>before</em> promoting the server to a Domain Controller.</div>

1. **Change Hostname:**
   * Open **Server Manager** > **Local Server**.
   * Click the default Computer Name (e.g., `WIN-XXXXXXX`).
   * Rename the computer to your preference, I renamed it to **`NYCE-DC01`** and restart the virtual machine.

   ![Rename the server](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Rename-the-Server.jpg)

2. **Configure Static IP Address:**
   * Press `Win + R`, type `ncpa.cpl`, and press **Enter**.
   * Right-click your network adapter > **Properties** > **Internet Protocol Version 4 (TCP/IPv4)**.
   * Manually assign a static IP address (e.g., IP: `192.168.120.100`, Subnet: `255.255.255.0`, Gateway: `192.168.120.1`, Preferred DNS: `127.0.0.1`).

   ![Configure-Static-IP](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Configure-Static-IP.jpg)


---

## Step 3: Installing & Promoting AD DS

### Phase A: Adding the AD DS Role
1. In **Server Manager**, click **Manage** > **Add Roles and Features**.

   ![Add Features](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Add-Features.jpg)
   
2. Progress through the wizard until reaching **Server Roles**.
3. Check **Active Directory Domain Services** (click **Add Features** on the pop-up).
4. Click **Install** and wait for completion.

### Phase B: Promoting to Domain Controller
1. Click the **Yellow Notification Flag** in Server Manager.
2. Select **Promote this server to a domain controller**.

   ![Promote to DC](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Promote-to-Domain-Controller.png)
   
3. Choose **Add a new forest** and enter your domain name:
   * **Root Domain Name:** `nycehomelab.local`
4. Set a strong **DSRM (Directory Services Restore Mode) Password**.
5. Leave remaining options at default and click **Install**. The server will automatically reboot.

---

## Step 4: Testing & Verification

1. Log into the server using domain administrator credentials: `NYCEHOMELAB\Administrator`.
2. Open **Server Manager** > **Tools**.
3. Verify the following tools are present and functional:
   - [x] Active Directory Users and Computers
   - [x] DNS Manager
   - [x] Group Policy Management

