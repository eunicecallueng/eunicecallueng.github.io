---
title: "Phase 2: Core Identity & Access Control (ADUC)"
date: 2026-07-05
categories: [Homelab, Active Directory]
tags: [aduc, rbac, user-management]
hidden: true
sitemap: false
permalink: /posts/ad-phase-2-identity/
---

## Now that the foundational Active Directory forest and client domain join are up and running, the next step is establishing structured identity management. In a real-world enterprise environment, you don't just dump every user and computer into a single default container; you organize them logically to streamline administrative control and security policies.

# **Step 1. Designing the Organizational Unit (OU) Structure**
## To mirror standard directory design best practices, I structured my domain (nycehomelab.local) with a dedicated organizational layout:

* Departmental OUs: Created logical containers (such as IT, HR, and Operations) to separate user accounts based on operational roles.

* Computer & Server OUs: Grouped machine accounts separately to allow targeted Group Policy deployment without affecting user configurations.

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/Dme0wktUKRY?si=DYlE5zvDcKxG4UQK" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# **Step 2:Domain Join & Client Network Configuration**

## Building on the foundation from Phase 1, I brought my client machine (my Windows 11 VM) into the newly minted environment. Before any machine can talk to a domain controller, getting the network and DNS pointing correctly is crucial:

   * **Configuring Client DNS:** I opened network properties on the Windows 11 client and pointed its Preferred DNS Server directly to my Domain Controller’s static IP address (``192.168.120.100``). This ensures the client can properly query and resolve the Active Directory domain.

      <iframe width="100%" height="400" src="https://www.youtube.com/embed/wu-0Izl8QDU?si=7WFvcqeab0K3i2NF" title="Client VM DNS coniguration" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

   * **Joining and Logging In:** After joining the machine to ``nycehomelab``.local, I tested the environment by logging into the client VM using a custom domain user account (``john.doe``) that I set up in Active Directory, verifying that centralized authentication was working seamlessly.

      <iframe width="100%" height="400" src="https://www.youtube.com/embed/lsE92TkJu_Y?si=BG69DJW2sgD6Rymm" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# **Step 3. User & Group Management via ADUC**
## With the container structure ready, I moved on to provisioning identities using Active Directory Users and Computers (ADUC):

* Standard & Privileged Users: Provisioned day-to-day testing accounts (like john.doe) alongside administrative service accounts.

* Security Groups: Established security groups to handle role-based access control rather than assigning permissions to individual users.

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/RhNyKZU3UoI?si=l2UnsT5igak09O7d" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

[← Back to AD Series Overview](/posts/active-directory-series/)