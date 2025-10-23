---
title: "Cicada HackTheBox machine walkthrough"
excerpt_separator: "<!--more-->"
date: 2024-10-02
categories:
  - HackTheBox Writeups
tags:
  - Hacking
  - Notes
  - writeup
  - ctf
  - AD
  - Windows
---


### Cicada — HackTheBox Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*

---

### Target Surface:

    - AD domain cicada.htb with DC CICADA-DC
    - SMB with guest access and shares DEV and HR
    - Credential gossip in HR share, subsequent user hunting via RID brute force
    - Remote Management access group membership for lateral movement
    - Backup Operators group enabling NTDS backup and hash extraction

### Tooling:

    - nmap for service discovery
    - smbclient for share enum and file retrieval
    - NetExec/CrackMapExec RID brute force
    - enum4linux for domain enumeration
    - evil-winrm for shell access
    - diskshadow, secretsdump for NTDS and SYSTEM extraction and dumping
    - hash cracking or pass-the-hash for admin access

---

*Excuse my sarcastic tone, that's how i brainstorm xD*

## Enumeration:

- After using nmap to look for open ports, its clearly shown that the box is in an Active Directory and the DC is CICADA-DC.
- The domain is cicada.htb.
- it’s also shown that smb port is open and allow guest login.
- listing the share using smbclient and the guest account:

```jsx
smbclient \\\\10.10.11.35\\ -U guest
```

- there are two open shares DEV and HR, DEV can be mapped but can not be listed with the current account while HR…
- HR share had a Note called “Notice from HR” it’s about welcoming the new hire without citing the name and putting their default password for anyone to see tsk tsk. The password is “Cicada$M6Corpb*@Lp#nZp!8”
- Now comes the real work, enumerating the domain users couldn’t be done with a bruteforce since we don’t have a wordlist so there’s a tool that did the trick- and no it’s better than crackmapexec.

```jsx
nxc smb 10.10.11.35 -u 'Guest' -p '' --rid-brute
```

- This would give us around 5 usernames so trying one by one with the given password we find that the owner is michael.wrightson
- using the now acquired creds with enum4linux.

```jsx
enum4linux -a -u "michael.wrigthson" -p "blabla" 10.10.11.35
```

- it lists a bunch of info about the domain but there was one noticeable note from “david.orelious”- you guessed right it’s his password.
- using again enum4linux with the new creds we find that david is the dev xD
- i immediately went back to the DEV share and bingo there’s a script with emily.oscars’ password- they never learn.
- Earlier when reading enum4linux report of michael i saw that emily is in a remote management access group of the domain which means we can connect remotely with her creds.
- Using evil-winrm that’s exactly what i did.

```jsx
evil-winrm -i 10.10.11.35 -u emily.oscars -p 'Q!3@Lp#M6b*7t*Vt'
```

## Foothold and User flag :

- Now that we’re in the cicada machine the user flag sits beautifully in emily’s desktop. “hidden”

## Root flag :

- I did some ldap dump to see things clearly, something shined differently, emily was part of a group called “backup operators”
- A quick search on google i found that they’re a critical asset in the domain they can backup, restore and even shutdown the machine etc so i knew that was the key for privilege escalation.
- i followed a walkthrough using that method in a different machine since it’s new to me from “https://snowscan.io/htb-writeup-blackfield/#”
- the first step is creating a “pwn.txt” file that contains the commands and upload it below:
    
![cicada-01](/assets/images/cicada-01.png)
    

- After creating a disk shadow of C:\ , we retrieve the ntds.dit containg the hashes.

![cicada-02](/assets/images/cicada-02.png)

- Those hashes need to be cracked so we also had to get a copy of system, then download both of them to my machine.

![cicada-03](/assets/images/cicada-03.png)

- The last step was dumping the hashes using secretsdump

```bash
secretsdump.py -ntds ntds.dit -system system local
```

![cicada-04](/assets/images/cicada-04.png)

- Now copying that administrator hash we use a pass-the-hash technique using evil-winrm and connect to the domain with the administrator creds.
- the root flag was in the desktop “hidden”

## Mistakes i made:

- Now don’t think that everything was straightforward, i struggled in the enumeration part because crackmapexec wasn’t working with the —rid-brute so I wasted time thinking the method was wrong.
- i also tried the kerberos_enumeration in metasploit to bruteforce using different username wordlists but that didn't.
- I got into a rabbit hole thinking it’s the right way while it was actually a whole different machine lol they’re using a forest and hosting the domains in different trees that’s why so i somehow got into the other tree with guest.
- it was an easy machine indeed tho, after i found the first username it was pretty straightforward.