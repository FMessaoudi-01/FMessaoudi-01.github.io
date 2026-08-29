---
layout: post
title: "SOC Detection Lab: Ongoing Notes"
date: 2025-12-10
categories: [Projects]
tags: [wazuh, sysmon, detection, active-directory,
       blue-team, defender]
---

## Lab Setup

- Wazuh SIEM connected to Windows AD environment
- Sysmon deployed for rich process/network telemetry
- Goal: practice detection engineering against known 
  attacker techniques

## Architecture

![architecture](/assets/images/soc-detection.png)

## Status

🟡 In progress — actively adding detection rules
and testing against AD attack scenarios.

## Detection Rules Built So Far

*(updated as I go)*

- [ ] Sysmon Event ID 1 — suspicious process creation
- [ ] Pass the Hash detection via Event ID 4624 type 3
- [ ] Mimikatz LSASS access patterns

## Resources I'm Following

- /

## Next Steps

- Map detections to MITRE ATT&CK
- Test against the AD attack chain from my red team lab