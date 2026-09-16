---
title: Wireshark Issue Simulations
date: 2026-06-08 21:00:00 +0800
categories: [Networking, Packet Analysis]
tags: [wireshark, pcap, troubleshooting, networking, protocols]
description: Turns out the best way to learn network troubleshooting is to break stuff on purpose! I set up Wireshark on my Windows 11 machine, deliberately messed up specific network configurations, and tracked the packet traffic in real time. It was an awesome hands-on experiment that helped me finally connect the dots between network errors and how to actually diagnose them.
permalink: /posts/wireshark-issue-simulations
pin : true
---

<div style="background: #11111b; border: 1px solid #313244; border-radius: 8px; padding: 18px; font-family: monospace;">
  <div style="color: #6c7086; margin-bottom: 12px; font-size: 0.9em;">$ select-module --wireshark-simulations</div>
  
  <div style="margin-bottom: 8px;">
    <span style="color: #89b4fa;">[01]</span> <a href="/posts/ws-1-packet-loss/" style="color: #cdd6f4; text-decoration: none;">Network Latency and Packet Loss</a>
  </div>
  <div style="margin-bottom: 8px;">
    <span style="color: #a6e3a1;">[02]</span> <a href="/posts/ws-2-dns-resolution-failure/" style="color: #cdd6f4; text-decoration: none;">DNS Resolution Failure</a>
  </div>
  <div style="margin-bottom: 8px;">
    <span style="color: #f9e2af;">[03]</span> <a href="/posts/ws-3-tcp-connection-reset/" style="color: #cdd6f4; text-decoration: none;">TCP Connection Reset</a>
  </div>
  <div>
    <span style="color: #f38ba8;">[04]</span> <a href="/posts/ws-4-http-vs-https/" style="color: #cdd6f4; text-decoration: none;">HTTP vs HTTPS</a>
  </div>
</div>