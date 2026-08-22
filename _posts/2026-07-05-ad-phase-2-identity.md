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

# **Step 2. User & Group Management via ADUC**
## With the container structure ready, I moved on to provisioning identities using Active Directory Users and Computers (ADUC):

* Standard & Privileged Users: Provisioned day-to-day testing accounts (like john.doe) alongside administrative service accounts.

* Security Groups: Established security groups to handle role-based access control rather than assigning permissions to individual users.

    <iframe width="100%" height="450" src="https://www.youtube.com/embed/RhNyKZU3UoI?si=l2UnsT5igak09O7d" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

[← Back to AD Series Overview](/posts/active-directory-series/)