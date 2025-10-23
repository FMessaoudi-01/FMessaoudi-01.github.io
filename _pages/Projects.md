---
layout: single
title: "Projects"
permalink: /projects/
toc: false
---

## Projects — Labs & Writeups

A curated list of my projects. Click any card to open the project hub with step-by-step notes.

{% assign all_projects = site.pages | where_exp: "p", "p.path contains 'projects/'" %}
{% assign projects = all_projects | where_exp: "p", "p.path contains 'index.md'" | sort: "title" %}

<div class="projects-grid">
  {% for p in projects %}
    <article class="project-card">
      <a class="card-link" href="{{ p.url | relative_url }}">
        {% if p.image %}
          <div class="card-thumb"><img src="{{ p.image | relative_url }}" alt="{{ p.title }}"></div>
        {% endif %}
        <div class="card-body">
          <h3>{{ p.title }}</h3>
          {% if p.description %}<p class="card-desc">{{ p.description }}</p>{% endif %}
          {% if p.date %}<time datetime="{{ p.date }}">{{ p.date | date: "%Y-%m-%d" }}</time>{% endif %}
        </div>
      </a>
    </article>
  {% endfor %}
  <article class="project-card">
  <a class="card-link" href="https://github.com/FMessaoudi-01/ERC20-using-foundry" target="_blank">
    <div class="card-thumb">
    </div>
    <div class="card-body">
      <h3>ERC20 token using Foundry</h3>
      <p class="card-desc">A simple ERC20 token project built with Foundry as part of my Web3 security learning path.</p>
    </div>
  </a>
</article>

<article class="project-card">
  <a class="card-link" href="https://github.com/FMessaoudi-01/Smart-Contract-FundMe" target="_blank">
    <div class="card-thumb">
    </div>
    <div class="card-body">
      <h3>FundMe smart contract project using solidity</h3>
      <p class="card-desc">A Foundry-based project that explores Solidity fundamentals — creating a decentralized funding contract with ETH value handling and withdrawal logic.</p>
    </div>
  </a>
</article>

</div>

