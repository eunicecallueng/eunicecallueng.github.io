---
title: "Phase 3: Storage & File Sharing"
date: 2026-07-05
categories: [Homelab, Active Directory]
tags: [file-server, smb, ntfs-permissions]
hidden: true
sitemap: false
permalink: /posts/ad-phase-3-storage/
---

[← Back to AD Series Overview](/posts/active-directory-series/)

<div style="background-color: #99cc33; color: #000000; padding: 20px; border-radius: 6px; margin-bottom: 20px;">
  <strong style="font-size: 1.1em;">💡 Best Practice Tip: Principle of Least Privilege (PoLP)</strong>
  <p style="margin-top: 10px; margin-bottom: 0;">When securing resources, it's best to configure broad access (like setting <em><strong>Everyone: Full Control</strong></em>) strictly at the <strong>Share Permissions</strong> level. You then restrict the actual granular access using <strong>NTFS Security ACLs</strong> tied to your AD Security Groups. This prevents conflicting permissions and keeps access clean!</p>
</div>