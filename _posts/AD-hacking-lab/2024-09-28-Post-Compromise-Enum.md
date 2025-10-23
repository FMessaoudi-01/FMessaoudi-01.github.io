---
layout: single
title: "Post-Compromise Enumeration in AD"
date: 2024-09-28
categories:
  - Active Directory
tags:
  - AD
  - Post-Compromise
permalink: /projects/ad-hacking-home-lab/post-compromise-enum/
order: 2
---

### ldapdomaindump :

- using kind of like the same method of mitm6 but with ldap

```jsx
sudo ldapdomaindump ldaps://192.168.52.138 -u 'RACCOON\credfield' -p Password1
```

### Bloodhound :

- it’s the same but richful GUI app, it gives so much details and links between the domain controller nodes.

```jsx
#start neo4j first
sudo neo4j console
#start bloodhound
sudo bloodhound
```

- Then generate the data using bloodhound-python tool

```jsx
sudo bloodhound-python -d RACCOON.local -u credfield -p Password1 -ns 192.168.52.138 -c all
```

- After the data is generated we just upload it to bloodhound app and view it as we like into graph

### Plumhound :

- neo4j and bloodhound need to be running before starting plumhound.
- make a connecction using this:

```jsx
sudo python3 PlumHound.py --easy -p Neo4j@2024
```

- it’s gonna generate some information by quering the domain, now we gotta dig deeper using the reporting tasks.

```jsx
sudo python3 PlumHound.py -x tasks/default.tasks -p Neo4j@2024
```

- that will give us so much information and we should gather as much as we can anc connect the dots.