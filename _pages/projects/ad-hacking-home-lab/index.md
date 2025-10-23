---
layout: single
title: "Active Directory Hacking — Home Lab"
permalink: /projects/ad-hacking-home-lab/
toc: false
---

Welcome — this is the central hub for my **Active Directory Hacking** home lab.  
Below are the organized notes & writeups for each step.

## Steps & Notes

{% assign project_folder = "_posts/AD-hacking-lab" %}
{% assign steps = site.posts | where_exp: "p", "p.path contains project_folder" | sort: "date" | reverse %}

<ul class="project-steps">
  {% for post in steps %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      {% if post.date %}<span class="muted"> — {{ post.date | date: "%Y-%m-%d" }}</span>{% endif %}
      {% if post.excerpt %}<p class="excerpt">{{ post.excerpt | strip_html | truncate: 160 }}</p>{% endif %}
    </li>
  {% endfor %}
</ul>
