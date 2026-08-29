---
layout: post
title: "Phishing via Sender Spoofing — Detection,
        Analysis and Response"
date: 2026-06-12
categories: [Incident Response, Phishing Analysis]
tags: [ir, phishing, spoofing, email-security,
       blue-team, dfir]
---

## Context

Several users reported
receiving a suspicious email appearing to originate from
a legitimate internal sender. Investigation revealed a
sender spoofing attack designed to harvest credentials
or deliver malicious content under the guise of a trusted
internal address.

This post documents the detection methodology, analysis
process, and response actions taken.

---

## How Sender Spoofing Works

Sender spoofing exploits the fact that SMTP does not
natively enforce sender identity. An attacker can set
the `From:` display field to any address they choose —
including a legitimate internal one without having
access to that mailbox.

The only reliable way to confirm spoofing is through
**raw email headers**, not the visible sender field.

Key headers to examine:

| Header | What it tells you |
|---|---|
| `Return-Path` | Actual address bounces go to and it should match From |
| `Received` chain | Full routing path of the email |
| `SPF` | Did it come from an authorized server? |
| `DKIM` | Was the message cryptographically signed? |
| `DMARC` | Did the domain policy flag or reject it? |

> **Note:** Forwarding a suspicious email loses the
> original routing headers. Always request the raw
> `.eml` file or access the mailbox directly.

---

## Detection

The incident surfaced through **user reports**, users
recognized the email as unexpected despite the familiar
sender address. This highlights a key principle:

> User awareness is a detection layer. Train people
> to report anomalies, not just avoid clicking.

Red flags that triggered further investigation:

- Sender address matched a known internal user but
  the email content was unexpected and out of context
- The email contained a zip file attachment with no
  prior communication context
- Subject line created urgency or imitated a system
  notification

---

## Analysis

### Step 1 — Raw Header Inspection

Retrieved the raw email headers and examined:

- `Return-Path` did not match the `From:` address
- `Received` chain showed the email originated from
  an external server with no relation to the
  organization's mail infrastructure

This confirmed spoofing, the email did not originate
from the displayed sender.

### Step 2 — Link / Attachment Analysis

The email contained a zip attachment and malicious IPs submitted to:
- **URLScan.io** — page structure and redirect analysis
- **VirusTotal** — URL reputation check
- **AbuseIPDB** — known phishing database lookup

Findings: the zip file pointed to multiple C2 servers (from VirusTotal hash Graph analysis)

### Step 3 — Scope Assessment

- Identified all users who received the email
- Determined which users interacted with the email

---

## Response Actions

**Immediate containment:**
- Alerted all users who received the email to not
  click and to not interact
- Enforced immediate password reset
- Compiled full user list for follow-up

**Escalation:**
- Incident reported to HQ security team with:
  - Raw header analysis
  - IOCs (spoofed sender, malicious URL, sending IP, C2 server IPs)
  - Affected user list
  - Recommended mail gateway block

**Recommended actions escalated:**
- Block sending domain/IP at mail gateway level
- Consider MFA enforcement to limit credential reuse
  impact

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Phishing | T1566.001 | Spearphishing via attachment |
| Social Engineering | T1684.001 | Impersonation |
| Valid Accounts | T1078 | Credential harvesting for account access |

---

## Key Takeaways

- Never confirm spoofing from the visible `From:` field: always pull raw headers
- Forwarding a suspicious email destroys the evidence: request the `.eml` or access the mailbox directly
- Legitimate platforms (Weebly, Google Sites) are
  increasingly used to host phishing pages specifically
  to bypass URL filters
- SPF/DKIM/DMARC misconfiguration is what makes
  spoofing possible: enforcement is the long-term fix
- User reporting is a detection layer

---

## Tools Used

| Tool | Purpose |
|---|---|
| MXToolbox | SPF / DKIM / DMARC verification |
| URLScan.io | URL and page analysis |
| VirusTotal | URL reputation |
| AbuseIPDB | Known phishing database |
| EmailRep.io | Sender reputation check |
| Raw header analyzer | Header parsing and routing analysis |