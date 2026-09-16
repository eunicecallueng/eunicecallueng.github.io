---
title: "Network Latency and Packet Loss"
date: 2026-07-07
categories: [Wireshark]
tags: [windows-server, vmware, domain-controller]
hidden: true
sitemap: false
permalink: /posts/ws-1-packet-loss/
---

I wanted to see how a degraded network connection (like bad Wi-Fi) looks at the packet level. To do this, I used a tool called **Clumsy** to deliberately inject lag and drop packets into my connection while targeting a specific IP address.

## **Simulation Steps**

1. I opened **Clumsy** as an administrator and set the filter to **`outboound and ip.DstAddr == 8.8.8.8`** to ensure I only messed with my traffic going tp 8.8.8.8
  
2. I turned on **Lag** and set it to **`500ms`**, and enabled **Drop** with a **`20%`** chance.
   
3. I started a live capture on **Wireshark** using the display filter **`ip.addr == 8.8.8.8`**.
   
4. From my terminal, I ran **`ping 8.8.8.8 -t`** to generate test traffic. I immediately saw some pings taking half a second while others timed out completely.

5. After the pings finished, I stopped the capture and turned off Clumsy.




---

### Wireshark Analysis & Filters
To isolate the affected traffic and observe the network's reaction to the loss, I used the following display filter:
```tcp.analysis.flags || icmp```

---


## Analysis & Findings
* **TCP Retransmissions:** Wireshark highlighted packets in red, indicating that the source had to resend data because the destination never acknowledged it.
* **Duplicate ACKs:** The receiver kept asking for the missing data packets that were dropped by the simulation.
* **ICMP Timeouts:** The ping requests showed dropped packets ("Request timed out") and significantly higher round-trip times (RTT).


---

## Want to check out the capture?
If you want to dive into the data yourself, just open the `.pcapng` file in Wireshark and use these display filters:
* To spot the duplicate ACKs: `tcp.analysis.duplicate_ack`
* To spot the retransmissions: `tcp.analysis.retransmission`
