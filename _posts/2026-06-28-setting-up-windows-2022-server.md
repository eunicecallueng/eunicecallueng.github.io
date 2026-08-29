---
title: "Setting up Windows 2022 Server in VMware Workstation"
date: 2026-06-28
categories: [Infrastructure, Virtualization]
tags: [vmware, windows-server, active-directory, homelab, networking]
description: >- 
   This is a step-by-step documentation of how I successfully installed Windows Server 2022 inside VMware Workstation Pro and promote it to a Domain Controller.
permalink: /posts/setting-up-windows-2022-server/
image:
   path: /assets/thumbnails/windows-2022-server-in-vmware.jpg
---
  


## Step 1: Prerequisites & Free Tools I Used

Before starting with Active Directory, you'll need to have your hypervisor ready and download the Windows Server ISO:

* **Hypervisor:** VMware Workstation Pro  
  *(If you haven't installed it yet, check out my other guide: [VMware Workstation Pro](https://github.com/eunicecallueng/Virtualization-Labs/blob/main/01.VMware-Workstation-Pro/README.md))
* **Operating System:** [Windows Server 2022 ISO](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) (Microsoft gives a free 180-day trial!)


---

## Step 2: Setting Up the Virtual Machine Container

Here is how I configured the VM settings in VMware before booting up the OS:

1. **Launch VMware Workstation** — Click on **Create a New Virtual Machine** (or hit `Ctrl + N`) to open the setup wizard.
2. **Choose Configuration Type** — Since this is just for my lab experiment, I went with **Typical (recommended)** and clicked **Next**.
3. **OS Installation Selection** — I selected **"I will install the operating system later"**.
 
   ![I will install the operating system later](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/install-os-later.jpg)

   > 💡 *Pro-Tip:* Doing this skips VMware's "Easy Install" feature, which often causes annoying errors during Windows Server installation!
4. **Select Guest OS** — I picked **Microsoft Windows**, and set the version to **Windows Server 2022**, then hit **Next**.
5. **Set VM Name & Location** — Name your VM whatever you like and click **Next**.
6. **Allocate Disk Space** — I set the maximum disk size to **30 GB** (which is plenty for a basic lab environment) and clicked **Next**.

   * For almost all modern homelab setups, **Store virtual disk as a single file** is the better option.  
      * Why Single File is Best?
         * Better Performance: Avoids file system fragmentation on your host drive. Reading/writing to a single contiguous block is cleaner, especially for IO-heavy VMs like Active Directory, SQL, or Kubernetes nodes.  
         * Easier File Management: Keeps your project directory clean. You won't have dozens of .vmdk chunks (e.g., s001.vmdk, s002.vmdk) cluttering your VM folder.Modern Storage Handles It: The main historical reason for splitting files (FAT32 file system's 4GB file limit) is obsolete. NTFS, APFS, and ext4 handle multi-terabyte files without issues.


7. **Attach the ISO File** — Before finishing, we need to make sure the VM actually boots from our downloaded Windows Server ISO:
   * Click **Customize Hardware...**
   
     ![Customize Hardware](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/Customize-Hardware.jpg)
     
   * Select **New CD/DVD (SATA)** from the left menu.
   * Under **Connection** on the right side, choose **Use ISO image file**.
   * Click **Browse...** and select your downloaded Windows Server 2022 `.iso` file.

     ![ISO Image file](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/ISO-image-file.jpg)
     
8. **Finish Up** — Click **Close**, then hit **Finish**!

---

## Step 3: Installing & Setting Up Windows Server 2022

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

   ![Windows 2022 Instance](https://raw.githubusercontent.com/eunicecallueng/Virtualization-Labs/main/02.Microsoft-Active-Directory/Configuring-Active-Directory/Screenshots/windows-2022-instance.jpg)


