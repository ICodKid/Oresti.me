---
title: Turning an Old MacBook into a Proxmox Server
description: How I salvaged a broken 2015 MacBook and turned it into a Proxmox-based homelab with TrueNAS, OpenVPN, and plans for Pi-hole.
slug: macbook-to-proxmox-server
published: 2025-09-11T10:00:00Z
updated: 2025-09-11T10:00:00Z
tags: [homelab, proxmox, truenas, openvpn,]
category: projects
draft: false
author: Oresti Tushe
canonical: https://oresti.me/blog/macbook-to-proxmox-server
image: ./Mac.png
---

# Turning an Old MacBook into a Proxmox Server

Recently I decided to breathe new life into my old 2015 MacBook Pro (16-inch) by turning it into a Proxmox server. The laptop had been dead for years. It would not boot when plugged in, and its screen had broken about five years earlier. I had given up on it until I decided to open it up and see if it could be revived.

---

## Step 1: Repairing and Cleaning the Laptop

Once I removed the bottom cover, I quickly discovered the problem. The inside was full of dust and gunk, and the battery was badly swollen. It was likely that the faulty battery had been causing short circuits, preventing the MacBook from starting.

:::warning
Lithium batteries can be dangerous. Handle them with extreme care. Swollen or damaged batteries should be properly recycled at an approved facility, never thrown in the trash.
:::

I disconnected the battery, removed the broken display and hinges, and cleaned every part I could, from the fans to reseating the SSD. To my surprise, the MacBook booted once the bad battery was out. Performance was slow due to years of bloat and startup apps, but it was running again and perfect for a homelab.

---

## Step 2: Installing Proxmox

With the hardware cleaned and stripped down, I created a bootable USB with [Proxmox](https://www.proxmox.com/en/) and booted into the installer. The setup went smoothly, although I made a small mistake at first by using the wrong router IP for DNS, which prevented package downloads. Once I corrected the settings, everything worked fine.

---

## Step 3: Building the Homelab

After Proxmox was up and running, I started creating virtual machines and containers for my homelab:

- **TrueNAS VM**: Allocated around 500GB of storage, created two SMB shares, and set up mirroring for redundancy. I also added tasks like caching and scheduled jobs to improve performance.  
- **OpenVPN Container**: Configured OpenVPN, set up port forwarding rules on my router, and enabled remote access to my files securely.  

This setup already gave me a reliable storage and VPN solution running on what was once a dead laptop.

---

## Step 4: Next Steps

I am planning to add **Pi-hole** as a DNS server to block ads and malicious domains across my network. This will bring an extra layer of security and improve the browsing experience for all devices connected to my home network.

---

## Conclusion

What started as a dead and forgotten MacBook turned into a surprisingly powerful little Proxmox server. By removing the broken battery, cleaning the insides, and installing Proxmox, I now have a homelab with TrueNAS storage, remote access through OpenVPN, and room for future improvements like Pi-hole.

This project showed me that with a bit of repair work and some creativity, old hardware can still be given a second life.

