---
layout: page
title: "Writeups"
permalink: /writeups/
toc: false
icon: fas fa-pen-to-square
order: 2
---

Technical writeups from labs, CTF platforms, and real-world
security work.

---

{% assign ir_posts = site.posts 
   | where_exp: "p", "p.path contains 'incident-response'" 
   | sort: "date" | reverse %}

### 🔵 Incident Response — Real World

{% assign ir_posts = site.posts | where_exp: "p",
"p.path contains 'incident-response'" | sort: "date" | reverse %}
{% for post in ir_posts %}
- **[{{ post.title }}]({{ post.url | relative_url }})** —
  <span style="color: var(--text-muted-color)">
  {{ post.date | date: "%Y-%m-%d" }}</span>
{% endfor %}

---

### 🔴 HackTheBox

{% assign htb_posts = site.posts | where_exp: "p",
"p.path contains 'hackthebox'" | sort: "date" | reverse %}
{% for post in htb_posts %}
- **[{{ post.title }}]({{ post.url | relative_url }})** —
  <span style="color: var(--text-muted-color)">
  {{ post.date | date: "%Y-%m-%d" }}</span>
{% endfor %}

---

### 🔴 TryHackMe

{% assign thm_posts = site.posts | where_exp: "p",
"p.path contains 'tryhackme'" | sort: "date" | reverse %}
{% for post in thm_posts %}
- **[{{ post.title }}]({{ post.url | relative_url }})** —
  <span style="color: var(--text-muted-color)">
  {{ post.date | date: "%Y-%m-%d" }}</span>
{% endfor %}