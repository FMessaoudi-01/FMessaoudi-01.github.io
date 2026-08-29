---
layout: post
title: "SOC Home Lab: Architecture and Setup"
date: 2024-10-01
categories: [Projects]
tags: [wazuh, thehive, shuffle, soar, siem]
---

## Overview

Built a fully functional SOC environment on Windows to 
simulate real-world threat detection and incident response
workflows.

## Stack

- **Wazuh** is a powerful tool that provides intrusion detection, integrity checking and Windows registry monitoring. In our setup, it was responsible for receiving events and sending alerts, including the detection of Mimikatz usage. 

- **TheHive** is a scalable, open-source and free Security Incident Response Platform which was used for managing security incidents. It received the enriched IOC data from VirusTotal and streamlined incident management.

- **Shuffle** is an automation platform that was instrumental in orchestrating the workflow. It connected the different tools and automated the response actions. 

- **Windows 10** served as the client machine in our setup. It was the system being monitored for potential security incidents. 

## Architecture

![architecture](/assets/images/soc-project.png)

