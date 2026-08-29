---
layout: post
title: "Reconstructing a 7-Year Dormant VBScript Worm
        from Forensic Artefacts"
date: 2026-04-23
categories: [Incident Response, Malware Analysis]
tags: [ir, malware, forensics, vbscript, mitre-attack,
       blue-team, worm]
---

## Context

This post documents how I reconstructed the full infection
chain of dormant VBscript worm from forensic artefacts alone, without the original
installer files being present on disk.

---

## Initial Findings

The malware arsenal was located entirely inside a hidden
folder designed to mimic the Windows Recycle Bin:
`C:\$.RECYCLEBIN\`

Files found:

| File | Status | Notes |
|---|---|---|
| svshost.exe | Present | Renamed wscript.exe — typosquatting svchost.exe |
| Skype.rar | 0kb — empty | Payload already executed |
| Vlce.rar | 0kb — empty | Payload already executed |
| [user]hostname.exe | Present | Machine-specific executable |
| 01-07 (blobs) | 0kb — empty | Assembled and executed |
| aa.exe | Absent | Self-deleted after execution |
| irsetup.exe | Absent | Self-deleted after execution |

---

## Forensic Significance of 0kb Files

The presence of empty files is evidence of prior execution.

The worm empties payload files after execution to cover its
tracks while keeping the file structure intact. The 0kb
files confirm the payload ran successfully at some point
in the past.

---

## Infection Chain (Reconstructed)

Reconstructed based on on-site artefacts and documented
behavior of the Win32.HLLW.Autoruner2 malware family:

[Infected USB] → User opens Skype.zip
→ irsetup.exe runs silently (self-deletes after)
→ Drops arsenal into C:$.RECYCLEBIN
→ run1.vbs launches svshost.exe
→ svshost.exe /e:VBScript.Encode Vlce.rar
→ VBS executes:
├── Registry Run keys for persistence
├── Startup folder shortcuts
├── Generates [username]hostname.exe
├── Copies arsenal to every inserted USB


---

## AV Evasion Technique — username+hostname.exe

One notable technique: the VBScript reads the machine's
`%USERNAME%` and generates an executable named after the victim username.

Result: every infected machine carries a unique executable
filename making signature-based detection impossible
across a fleet.

---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Boot/Logon Autostart — Registry Run Keys | T1547.001 | Persistence via HKLM Run keys |
| Replication Through Removable Media | T1091 | USB propagation |
| Hide Artifacts — Hidden Files | T1564.001 | $.RECYCLEBIN folder |
| Obfuscated Files | T1027 | VBScript.Encode encoding |

---

## IOCs

| Type | Value | Source |
|---|---|---|
| Malware family | Win32.HLLW.Autoruner2.21083 | Dr.Web |
| Alternate name | Trojan.VBS.Agent.ahv | Kaspersky / MetaDefender |
| File path | C:\$.RECYCLEBIN\ | On-site |
| Persistence key | HKLM\...\Run — Manual, Printer, System | On-site |

*Hashes withheld — available upon request for verified
security professionals.*

---

## Key Takeaways

- 0kb files are forensic evidence, not noise
- Self-deleting installers don't erase the infection chain
- USBFix logs from 2017 provided the infection timeline
- Per-machine executable naming evades AV signature matching
- Always check `$.RECYCLEBIN` — distinct from `$Recycle.Bin`