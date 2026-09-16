---
title: "DNS Resolution Failure"
date: 2026-07-07
categories: [Wireshark]
tags: [windows-server, vmware, domain-controller]
hidden: true
sitemap: false
permalink: /posts/ws-2-dns-resolution-failure/
---

In this scenario, I simulated a **DNS failure**, which is a common network issue where **client machines cannot resolve domain names into IP addresses**.

---

## **Simulation Steps**
1. I manually broke the DNS configuration on Windows 11 using the following steps:
   * Opened **Settings** > **Network & internet** > **Wi-Fi** (or Ethernet).
   * Clicked on the network properties and found **DNS server assignment**, then clicked **Edit**.

     ![Network and Internet](./network_settings.png)

   * Changed it from **Automatic (DHCP)** to **Manual**, toggled on **IPv4**, and set the Preferred DNS to a fake, non-existent IP address (`10.0.0.99`), then clicked **Save**.
  
     ![Fake IP](./fake_IP.png)

2. I executed **`ipconfig /flushdns`** in the command prompt to clear the local resolver cache and force the OS to request fresh resolution.
3. I started a **Wireshark** capture and attempted to query a domain using **`nslookup github.com`**.

      ![Powershell](./flushdns.png)
  
4. After capturing the failing packets, I reverted my network adapter back to Automatic (DHCP).

---

## **Analysis & Findings**
To isolate the DNS traffic and analyze the failure pattern, I applied this display filter:
**``dns``**

* **Standard Queries with No Responses:** Wireshark showed **`Standard query 0x... A github.com`** outbound packets leaving my machine, but they were met with absolute silence.

    ![DNS response missing](./dns_response_missing.png)

* **Background Process Activity:** I noticed that various background applications (like **Skype** and other system services) were constantly attempting to reach their servers. Since my fake DNS server (`10.0.0.99`) was not responding, these apps spammed the network with DNS queries.
* **Retransmissions:** Because no DNS reply was received, the operating system and the background apps performed multiple **retransmissions**. You can see the same query ID being sent repeatedly at increasing intervals as the applications tried to recover from the lack of a response.

    ![DNS retransmission](./dns_retransmission.png)

    ngek
