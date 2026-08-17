---
title: "AD DS Installation & DC Promotion"
date: 2026-08-17 10:00:00 +0800
categories: [Homelab, Active Directory]
tags: [windows-server, active-directory, dns]
---

Inilatag ko ang aking core homelab forest (`nycehomelab.local`) sa pamamagitan ng pag-promote ng primary server (`NYCE-DC01`) bilang Domain Controller.

### Project Highlights
* **Domain Controller:** Promoted primary server `NYCE-DC01` at nag-build ng malinis na OU structure para sa users at devices.
* **High Availability & DNS:** Natutunan ko na sa totoong enterprise networks, laging dalawa o higit pa ang DCs para sa fault tolerance at gumagamit ng routable domain names.

🔗 **GitHub Repository:** [Tingnan ang Code sa GitHub](https://github.com/eunicecallueng/nycehomelab)
