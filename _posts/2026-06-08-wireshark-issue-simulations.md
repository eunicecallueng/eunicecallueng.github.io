---
title: Wireshark Issue Simulations
date: 2026-06-08 21:00:00 +0800
categories: [Networking, Packet Analysis]
tags: [wireshark, pcap, troubleshooting, networking, protocols]
description: Turns out the best way to learn network troubleshooting is to break stuff on purpose! I set up Wireshark on my Windows 11 machine, deliberately messed up specific network configurations, and tracked the packet traffic in real time. It was an awesome hands-on experiment that helped me finally connect the dots between network errors and how to actually diagnose them.
permalink: /posts/wireshark-issue-simulations
pin : true
---

<div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 16px; margin-top: 20px;">

  <!-- Phase 1 Card -->
  <a href="/posts/ws-1-packet-loss/" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid #333; border-radius: 8px; padding: 18px; background-color: #1e1e2e;">
      <h3 style="margin-top:0; color: #89b4fa;">Network Latency and Packet Loss</h3>
    </div>
  </a>

  <!-- Phase 2 Card -->
  <a href="/posts/ws-2-dns-resolution-failure/" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid #333; border-radius: 8px; padding: 18px; background-color: #1e1e2e;">
      <h3 style="margin-top:0; color: #a6e3a1;">DNS Resolution Failure</h3>
    </div>
  </a>

  <!-- Phase 3 Card -->
  <a href="/posts/ws-3-tcp-connection-reset/" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid #333; border-radius: 8px; padding: 18px; background-color: #1e1e2e;">
      <h3 style="margin-top:0; color: #f9e2af;">TCP Connection Reset</h3>
    </div>
  </a>

  <!-- Phase 4 Card -->
  <a href="/posts/ws-4-http-vs-https" style="text-decoration: none; color: inherit;">
    <div style="border: 1px solid #333; border-radius: 8px; padding: 18px; background-color: #1e1e2e;">
      <h3 style="margin-top:0; color: #f38ba8;">HTTP vs HTTPS</h3>
    </div>
  </a>

</div>