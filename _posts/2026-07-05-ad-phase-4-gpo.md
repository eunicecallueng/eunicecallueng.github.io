---
title: "Phase 4: Group Policy & Advanced Security (GPO)"
date: 2026-07-14
categories: [Homelab, Active Directory]
tags: [gpo, security, active-directory]
hidden: true
sitemap: false
permalink: /posts/ad-phase-4-gpo/
---

Welcome to the final phase of my Active Directory homelab project! After setting up the foundation, organizing users into logical OUs, and securing file shares, I turned my attention to enterprise-level automation and security baselines. 

In this phase, I focused on using Group Policy Objects (GPOs) and Active Directory's advanced security tools to lock down endpoints, automate user workflows, and secure identity accounts. Here is a walkthrough of how I built and tested these configurations in my lab.


## **Step 1: Establishing the GPO Central Store**
Before rolling out policies, it is a best practice to configure a Central Store. This ensures that all domain controllers pull from a single, up-to-date repository of Administrative Templates **(ADMX files)** when managing policies:
    
<iframe width="100%" height="450" src="https://www.youtube.com/embed/OYzf86gSntM?si=UGB45ufjPKrGIraC" title=" Central Store" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 2: Establishing Baseline Domain Security**
Once the Central Store was in place, I configured core domain security policies within the Group Policy Management Console (GPMC). I focused on defining baseline authentication settings, password rules, and account lockout policies across the entire domain.

<iframe width="560" height="315" src="https://www.youtube.com/embed/jw0WRVFzemM?si=enbWX6A0mj2UgyYP" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

---

## **Step 3: Automating Resources with Drive Mapping**
Rather than manually mapping network drives on every user's PC, I used GPO Preferences to automate the entire process. Now, when users log in, their assigned department share (which I built in Phase 3) maps automatically to their workstation based on their group membership.
(Insert your Drive_Mapping.mp4 and Shared Drive Mapping.mp4 embed codes here)

---

## **Step 4: Hardening Workstations & Securing Endpoints**
Locking down endpoint devices was one of the most hands-on parts of this lab. I created targeted GPOs to minimize the attack surface on domain workstations:

* **Removable Storage Blocking:** Configured policies to block unauthorized USB drives, preventing data exfiltration and external malware threats.
* **Credential Security:** Configured policies to restrict cached logon entries and protect credentials stored on local machines.
* **User Environment Hardening:** Restricted access to system settings, administrative tools, and unauthorized command interfaces for standard users.
* **Policy Enforcement & Testing:** Ran `gpupdate /force` across domain nodes and tested authentication rules on client VMs to make sure policies applied instantly without issues.
(Insert your GPO Removable Storage Blocking.mp4, GPO_Credential_Security.mp4, GPO_User_Environment_Security.mp4, and Workstation Hardening.mp4 embed codes here)

---

## **Step 5: Advanced Identity Security & Delegation**
To finish the phase, I implemented Active Directory's native identity protection features:

* **Fine-Grained Password Policies (FGPP):** Created stricter password requirements specifically for administrative users without impacting standard employees.
* **Logon Restrictions:** Applied time-based restrictions to prevent certain accounts from authenticating outside standard operational hours.
* **Delegation of Control:** Used the Delegation of Control Wizard to give the Helpdesk group permission to reset passwords without giving them full Domain Admin privileges.
(Insert your Fine-Grained Password Policies (FGPP).mp4, Restricting User Logon Hours.mp4, and Delegate Control.mp4 embed codes here)

---

## **Project Wrap-Up**
Building this 4-phase Active Directory lab from scratch—from a raw Server 2022 installation to a fully secured, automated enterprise architecture—was an incredible hands-on journey. It gave me deep practical experience in system administration, identity management, and network security baselines!