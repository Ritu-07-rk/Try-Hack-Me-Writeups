# 🪟 TryHackMe - Investigating Windows

> **Room:** Investigating Windows  
> **Platform:** TryHackMe  
> **Difficulty:** Easy  
> **Category:** Windows Forensics | Incident Response | DFIR

---

# 📖 Overview

The **Investigating Windows** room focuses on performing a forensic investigation of a compromised Windows Server. Throughout the investigation, multiple Windows artifacts are analyzed to determine attacker activity, persistence mechanisms, privilege escalation, lateral movement, and evidence of compromise.

Rather than using third-party forensic software, the investigation primarily relies on native Windows tools such as Event Viewer, Task Scheduler, Registry Editor, and PowerShell.

---

# 🎯 Learning Objectives

- Understand important Windows forensic artifacts.
- Investigate user logon activity.
- Analyze Windows Event Logs.
- Detect persistence mechanisms.
- Investigate Scheduled Tasks.
- Identify malicious PowerShell activity.
- Detect privilege escalation.
- Discover attacker infrastructure.
- Analyze DNS poisoning.
- Perform basic DFIR investigations.

---

# 🛠 Tools Used

- Event Viewer
- Computer Management
- Task Scheduler
- Windows Registry
- PowerShell
- Command Prompt
- Services.msc
- Local Users and Groups
- DNS Cache
- Windows Logs

---

# Task 1 - Windows Version

## Objective

Identify the operating system running on the compromised machine.

### Explanation


The version can be found using:

Go to settings > System > About

![Question1](/photos/q1.png)

### Question

**What's the version and year of the Windows machine?**

### Answer

```
Windows Server 2016
```

---

# Task 2 - Last Logged-on User

## Objective

Determine who last logged into the machine.

### Explanation

Relevant Event ID:

- 4624 → Successful Logon

Use the command: ```Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4624
} | Where-Object {
    $_.Properties[8].Value -in 2, 10
} | Select-Object -First 1 | ForEach-Object {
    [PSCustomObject]@{
        TimeCreated = $_.TimeCreated
        Username = $_.Properties[5].Value
        Domain = $_.Properties[6].Value
        LogonType = $_.Properties[8].Value
    }
}```

![Question2](/photos/q2.png)

### Question

**Which user logged in last?**

### Answer

```
Administrator
```

---

# Task 3 - John's Last Logon

## Objective

Determine when user John last authenticated.

### Explanation

Use this command: ``` Get-WinEvent -FilterHashtable @{LogName='Security'; Id=4624} |
Where-Object {
    $_.Properties[5].Value -eq 'John'
} |
Sort-Object TimeCreated -Descending |
Select-Object -First 1 TimeCreated,
@{Name='User';Expression={$_.Properties[5].Value}},
@{Name='Domain';Expression={$_.Properties[6].Value}},
@{Name='LogonType';Expression={$_.Properties[8].Value}} ```

![Question3](/photos/q3.png)

### Question

**When did John log onto the system last?**

### Answer

```
03/02/2019 5:48:32 PM
```

---

# Task 4 - Startup Network Connection

## Objective

Identify the first IP contacted after Windows booted.

### Explanation

When we start the window, a programs run in which the IP is visible.

![Question4](/photos/q4.png)

### Question

**What IP does the system connect to when it first starts?**

### Answer

```
10.34.2.3
```

---

# Task 5 - Administrative Accounts

## Objective

Identify all users with administrative privileges.

### Explanation

Use this command: ```Get-LocalGroupMember -Group "Administrators"```

![Question5](/photos/q5.png)

### Question

**What two accounts had administrative privileges (other than Administrator)?**

### Answer

```
Guest, Jenny
```

---

# Task 6 - Malicious Scheduled Task

## Objective

Identify persistence established through Windows Task Scheduler.

### Explanation

Open the task scheduler and check the tasks scheduled. 

![Question6](/photos/q6.png)

### Question

**What's the name of the scheduled task that is malicious?**

### Answer

```
Clean file system
```

---

# Task 7 - Malicious Script

## Objective

Identify the script executed by the scheduled task.

### Explanation

Check the actions section of the identified suspicious task in Task Scheduler.

![Question7](/photos/q7.png)

### Question

**What file was the task trying to run daily?**

### Answer

```
nc.ps1
```

---

# Task 8 - Listening Port

## Objective

Identify the local listening port created by the malicious script.

### Explanation

Check the actions section of the identified suspicious task in Task Scheduler.

![Question8](/photos/q8.png)

### Question

**What port did this file listen locally for?**

### Answer

```
1348
```

---

# Task 9 - Jenny's Last Logon

## Objective

Determine whether Jenny ever authenticated.

### Explanation

Windows stores the last successful logon time for every local account.

Use this command:  ```net user Jenny | findstr Last```

![Question9](/photos/q9.png)

### Question

**When did Jenny last logon?**

### Answer

```
Never
```

---

# Task 10 - Date of Compromise

## Objective

Identify the day attacker activity began.

### Explanation

Check the file modification dates.

![Question10](/photos/q10.png)

### Question

**At what date did the compromise take place?**

### Answer

```
03/02/2019
```

---

# Task 11 - Privilege Escalation

## Objective

Determine when special privileges were first assigned.

### Explanation

Security Event ID:

```
4635
```

This event indicates:

> Special privileges assigned to new logon

It often appears immediately after an administrator or attacker logs in.

![Question11](/photos/q11.png)

### Question

**During the compromise, at what time did Windows first assign special privileges to a new logon?**

### Answer

```
03/02/2019 4:04:49 PM
```

---

# Task 12 - Password Dumping Tool

## Objective

Identify the credential dumping tool.

### Explanation

Open the file location of the program which executed on startup. Check the output file and identify the credential dumpong tool.

![Question12](/photos/q12.png)

### Question

**What tool was used to get Windows passwords?**

### Answer

```
Mimikatz
```

---

# Task 13 - Command & Control Server

## Objective

Identify the attacker's external infrastructure.

### Explanation

Open the file location: `C:\ > Windows > System32 > drivers > etc > hosts `

There are two google mappings. Use the command "ping". If the IP matches to the one in host file then it is legitimate. Else, it may be used as C2 server.

![Question13](/photos/q13.png)

### Question

**What was the attacker's external control and command server IP?**

### Answer

```
76.32.97.132
```

---

# Task 14 - Uploaded Web Shell

## Objective

Identify the uploaded web shell extension.

### Explanation

Web shells provide attackers remote access through web servers.

Common extensions include:

- .php
- .aspx
- .jsp

  Go to : `C:/inetpub/wwwroot/`. Search about the file extensions on google.

  ![Question14](/photos/q14.png)

### Question

**What was the extension name of the shell uploaded via the server's website?**

### Answer

```
.jsp
```

---

# Task 15 - Final Opened Port

## Objective

Identify the last port opened by the attacker.

### Explanation

Attackers often open new ports to:

- bypass firewalls
- establish persistence
- create remote shells

Search the inbound rules in the Windows Firewall. Check the rules without any group.

![Question15](/photos/q15.png)

### Question

**What was the last port the attacker opened?**

### Answer

```
1337
```

---

# Task 16 - DNS Poisoning

## Objective

Determine which domain was targeted.

### Explanation

DNS poisoning redirects legitimate domains to malicious IP addresses.

Investigators should always inspect:

- Hosts file
- DNS Cache
- Registry
- Network configuration

Go to the hosts file opened earlier. The domain with varying IP is the answer.

### Question

**Check for DNS poisoning, what site was targeted?**

### Answer

```
google.com
```

---


# 🛡 MITRE ATT&CK Techniques

| Technique | ID |
|------------|------|
| Scheduled Task | T1053.005 |
| PowerShell | T1059.001 |
| Valid Accounts | T1078 |
| Credential Dumping | T1003 |
| Command and Control | T1071 |
| Web Shell | T1505.003 |
| DNS Manipulation | T1565 |

---

