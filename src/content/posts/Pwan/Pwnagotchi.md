---
title: Building and Testing My Pwnagotchi
description: My journey building a Pwnagotchi …
slug: building-and-testing-my-pwnagotchi
published: 2025-09-10T10:00:00Z
updated: 2025-09-10T10:00:00Z
tags: [Networks, Pwnagotchi]
category: projects
draft: false
author: Oresti Tushe
canonical: https://oresti.me/blog/building-and-testing-my-pwnagotchi
image: ./pwn.jpg
---

# Building and Testing My Pwnagotchi

I have always been curious about wireless security and wanted a project that let me really get my hands dirty with network security.Pwnagotchi a AI powered tool that listens for WPA handshakes and it felt like the perfect mix of hardware tinkering, Linux and ethical hacking.

---

## Step 1: Setting up the environment

Before I could even think about flashing an image, Getting my Windows machine ready. That meant downloading drivers, installing updates and making sure the Raspberry Pi Zero W would be recognised properly. Once that was sorted, I grabbed the official image here:

::github{repo="jayofelony/pwnagotchi"}

---

## Step 2: Installing plugins and configurations

My favorit part of a Pwnagotchi is how much you can customise it. I followed this guide to get plugins and configs set up:

::github{repo="SHUR1K-N/Project-Pwnag0dchi"}

Along the way I learned how to edit YAML configs, how to add plugins that show useful stats and how to troubleshoot when Windows did not want to cooperate learing to add and edit filis with sftp and contecting to it with ssh.

---

## Step 3: Real life testing

When the device finally booted and ran properly, I took it into a test environment where I had permission to use it. During one of the sessions the Pwnagotchi captured more than two thousand packets including multiple WPA handshakes.

:::caution
Always use your Pwnagotchi only on networks you own or ones where you have explicit permission to test. Capturing data without consent is illegal and unethical.
:::

The little e-ink display was a highlight. It reacted with different “faces” as packets came in. That made the process surprisingly fun while at the same time teaching me how much things like movement, signal strength and patience matter when collecting packets.

---

## Step 4: Cracking the captured files

Before showing the tools, it helps to understand what is actually happening. When a device connects to WiFi it goes through what is called the WPA2 four way handshake. In this exchange the client and the access point prove they both know the WiFi password without sending it directly. If the handshake is captured, it can be tested offline. If the password is weak, tools like Hashcat can guess it by running through wordlists until the right one matches.

Catching handshakes is only the first half of the process. To go further I had to convert the capture into a format Hashcat could actually work with. For that I used hcxtools:

::github{repo="ZerBea/hcxtools"}

```bash title="Convert capture to Hashcat format"
hcxpcapngtool -o capture.hc22000 -E essidlist.txt -I identitylist.txt capture.pcapng
```

Once converted I moved on to Hashcat itself. I tested it using a public wordlist from SecLists:

::github{repo="danielmiessler/SecLists"}

```bash title="Run Hashcat with a public wordlist"
hashcat -m 22000 capture.hc22000 /path/to/wordlists/rockyou.txt --status --status-timer=10 --force
```

:::caution
Only run cracking attempts on networks you own or are authorised to test. Doing this on someone else’s network without permission is against the law.
:::

On a test WiFi with a weak password Hashcat cracked it quickly. Watching that happen made it very clear why strong and unique passwords are essential for WiFi security.

---

## Step 5: What I learned

This project gave me a lot of practical knowledge. I figured out how to prepare Windows with the right drivers and networking setup. I learned how to flash an image safely to an SD card. I experimented with plugins to make the Pwnagotchi more useful. I saw how WPA handshakes are captured and how they can be tested with tools like Hashcat.

More than that, it reminded me that ethical responsibility matters just as much as technical skill. Tools like this are powerful and should only ever be used for learning and for security testing where you have permission.

---

## Conclusion

Building my Pwnagotchi was one of the most enjoyable and educational projects I have done so far. From the first driver update on Windows all the way to watching a weak test password fall to Hashcat, the process taught me new skills and gave me a deeper respect for cybersecurity.

If you want to try it yourself, start with the repos above and keep your testing limited to networks you own or are allowed to audit. You will come away with more than just a fun little gadget. You will gain knowledge and experience that can carry over into ethical hacking and penetration testing.
