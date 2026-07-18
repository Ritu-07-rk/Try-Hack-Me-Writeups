# 36 Hours of Rampage – Windows Forensics Investigation

![Forensics](https://img.shields.io/badge/Category-Windows%20Forensics-blue)
![Difficulty](https://img.shields.io/badge/Difficulty-Easy--Intermediate-green)
![Platform](https://img.shields.io/badge/Platform-TryHackMe-red)

---

# Overview

This room focuses on investigating a compromised Windows workstation by analysing various Windows forensic artifacts. The objective is to reconstruct the attacker's activities, identify the tools executed, determine which files were accessed, and uncover any anti-forensic actions performed to cover tracks.


# Scenario

Cybertees Pvt Ltd's HR employee, **James**, has a habit of writing sensitive information such as passwords on sticky notes and leaving them around his workstation.

After returning to work on Monday morning, James noticed:

- Important files were missing.
- Suspicious tools had been installed.
- Evidence of activity appeared to have been removed.

An internal investigation revealed CCTV footage showing another employee, **Johny**, accessing the workstation over the weekend.

Additional findings indicated:

- Johny had recently resigned.
- He was planning to join a competing company.
- Sensitive documents may have been accessed.
- Several files and tools had been deleted after use.

As a forensic investigator, the objective is to reconstruct Johny's activities during the **36-hour compromise window**.

---

# Learning Objectives

By completing this investigation, you will learn:

- Windows Registry Forensics
- Registry Hive Analysis
- User Activity Reconstruction
- LNK File Analysis
- ShellBag Investigation
- Jump List Analysis
- Identifying Anti-Forensic Techniques
- Building an Attack Timeline
- Mapping Findings to MITRE ATT&CK

---

# Tools Used

| Tool | Purpose |
|--------|----------|
| Registry Explorer | Analyze Registry Hives |
| EZ Tools Suite | Windows Artifact Analysis |
| LECmd | LNK File Analysis |
| ShellBag Explorer | ShellBag Parsing |
| JumpList Explorer | Jump List Analysis |

---

# Windows Registry Overview

The Windows Registry is a hierarchical database used to store configuration settings and options for both the operating system and installed applications.

It contains valuable forensic evidence regarding:

- User activity
- Program execution
- File access
- Device connections
- Software installations
- Search history

---

## Registry Hive Locations

Located primarily at:

```text
C:\Windows\System32\Config
```

### Important Registry Hives

| Hive | Mapped Key | Purpose |
|--------|------------|----------|
| SAM | HKLM\SAM | User Accounts |
| SECURITY | HKLM\SECURITY | Security Policies |
| SYSTEM | HKLM\SYSTEM | Hardware & Startup |
| SOFTWARE | HKLM\SOFTWARE | Installed Applications |
| DEFAULT | HKU\.DEFAULT | Default User Settings |
| NTUSER.DAT | HKCU | User-specific Settings |
| USRCLASS.DAT | HKCU\Software\Classes | User Shell Information |

---

# Task 3 – Revisiting Registry Artifacts

## Question 1

### Which Hive stores information about installed software?

#### Answer

```text
SOFTWARE
```

---

## Question 2

### What is the size of the SAM Hive?

#### Answer

```text
128 KB
```

---

# Task 4 – Registry Forensics

---

# TypedPaths Analysis

## Registry Location

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
```

TypedPaths records folders manually entered into Windows Explorer.

This artifact helps investigators identify:

- Directly accessed folders
- Hidden directories
- Suspicious locations

---

## Finding

The suspect manually entered a suspicious temporary directory.

### Answer

```text
C:\system\home\tmp
```

### Significance

This location later appears multiple times during the investigation, indicating it was used as a working directory.

---

# WordWheelQuery Analysis

## Registry Location

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\WordWheelQuery
```

WordWheelQuery stores search terms entered into Windows Explorer.

---

## Finding

Latest search term:

```text
wipe
```

### Significance

This suggests the attacker was actively searching for methods or tools to erase evidence from the system.

### Answer

```text
wipe
```

---

# ComDlg32 Analysis

## Registry Location

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32
```

Contains:

- LastVisitedMRU
- OpenSaveMRU

These artifacts reveal:

- Recently opened files
- Recently saved files
- Folder navigation history

---

## Finding

The last text file saved by the attacker was:

```text
C:\system\home\tmp\code.txt
```

### Answer

```text
C:\system\home\tmp\code.txt
```

### Significance

This indicates the attacker created or modified a file during the intrusion.

---

# UserAssist Analysis

## Registry Location

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
```

UserAssist tracks GUI application execution.

It records:

- Program Names
- Execution Counts
- Last Execution Time

---

## Suspicious Tool #1

### Executed Five Times

```text
keylogger.exe
```

### Significance

The attacker repeatedly executed a keylogging utility, likely to harvest credentials and monitor user activity.

### Answer

```text
keylogger.exe
```

---

## Suspicious Tool #2

### Disk Wiping Utility

```text
DiskWipe.exe
```

### Significance

This strongly indicates anti-forensic activity intended to destroy evidence.

### Answer

```text
DiskWipe.exe
```

---

# ShellBag Analysis

ShellBags store historical folder access information.

Even if folders are deleted, ShellBags often preserve evidence.

---

## Registry Files

```text
NTUSER.DAT
USRCLASS.DAT
```

---

## Registry Locations

```text
HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\BagMRU

HKCU\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\Bags
```

---

## Information Stored

- Folder Paths
- Network Shares
- Deleted Folders
- Timestamps
- User Preferences

---

## Network Share Investigation

### IP Address

```text
10.10.17.228
```

### Answer

```text
10.10.17.228
```

### Significance

The attacker accessed a remote network share containing sensitive company information.

---

## Network Folder Structure

Documents Folder:

```text
Documents
└── secret-doc
```

### Second Subfolder

```text
secret-doc
```

### Answer

```text
secret-doc
```

### Significance

This folder likely contained sensitive documents targeted during the intrusion.

---

# LNK File Analysis

---

## What Are LNK Files?

Windows automatically creates shortcut files whenever a user accesses:

- Files
- Applications
- Network Resources

---

## Common Locations

```text
%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Recent

%USERPROFILE%\Recent
```

---

## Tool Used

```powershell
LECmd.exe -f C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\code.lnk
```

---

## Information Extracted

LNK files reveal:

- File Name
- File Path
- Access Time
- Network Share
- File Size

---

## Finding

Recently accessed file:

```text
C:\system\home\tmp\code.txt
```

### Answer

```text
C:\system\home\tmp\code.txt
```

---

# Jump List Analysis

Jump Lists maintain records of:

- Frequently opened files
- Recently accessed documents
- Browser activity
- Application usage

---

## Locations

```text
%APPDATA%\Microsoft\Windows\Recent\AutomaticDestinations

%APPDATA%\Microsoft\Windows\Recent\CustomDestinations
```

---

# Browser Activity

### URL Accessed

```text
http://10.10.17.228/
```

### Answer

```text
http://10.10.17.228/
```

### Significance

Confirms direct access to the network share identified in ShellBag analysis.

---

# Sensitive Document Access

## File Accessed

```text
How to Hack.pdf
```

## Access Timestamp

```text
2024-03-04 12:28:26
```

### Answer

```text
2024-03-04 12:28:26
```

### Significance

Demonstrates the attacker accessed potentially sensitive or malicious instructional material.

---

# Indicators of Compromise (IOCs)

| Category | Indicator |
|------------|-----------|
| Directory | C:\system\home\tmp |
| Search Term | wipe |
| Keylogger | keylogger.exe |
| Wiper Tool | DiskWipe.exe |
| Network Share | 10.10.17.228 |
| Sensitive Folder | secret-doc |
| Browser URL | http://10.10.17.228/ |
| Accessed File | How to Hack.pdf |
| Saved File | code.txt |

---

# Attack Timeline

## Initial Access

Johny gains access to James' workstation.

---

## Discovery

Navigates through:

```text
C:\system\home\tmp
```

using Windows Explorer.

---

## Credential Collection

Executes:

```text
keylogger.exe
```

five times.

---

## Collection

Accesses network share:

```text
10.10.17.228
```

and browses:

```text
Documents\secret-doc
```

---

## Data Access

Opens:

```text
How to Hack.pdf
```

and other sensitive files.

---

## Staging

Creates or modifies:

```text
code.txt
```

within the temporary directory.

---

## Anti-Forensics

Searches:

```text
wipe
```

inside Explorer.

Executes:

```text
DiskWipe.exe
```

to remove traces.

---

## Cleanup

Deletes files and tools in an attempt to destroy evidence.

---

# MITRE ATT&CK Mapping

| Technique | ID |
|------------|------|
| Valid Accounts | T1078 |
| File and Directory Discovery | T1083 |
| Network Share Discovery | T1135 |
| Data from Network Shared Drive | T1039 |
| Data Staged | T1074 |
| Input Capture: Keylogging | T1056.001 |
| Indicator Removal on Host | T1070 |
| File Deletion | T1070.004 |

---

# Key Takeaways

- Registry artifacts reveal extensive user activity.
- UserAssist identifies executed GUI applications.
- TypedPaths and WordWheelQuery expose attacker intent.
- ShellBags retain evidence even after deletion.
- LNK files help reconstruct accessed files.
- Jump Lists provide a history of document and application usage.
- Correlating multiple artifacts enables accurate attack reconstruction.
- Anti-forensic actions can often be detected despite deletion attempts.

---

# Conclusion

The investigation successfully reconstructed the activities performed by Johny during the 36-hour compromise period. Evidence showed that he navigated through suspicious directories, executed a keylogger, accessed confidential data from a network share, interacted with sensitive documents, and finally attempted to erase evidence using a disk wiping utility.

Although files and tools were deleted, Windows forensic artifacts preserved enough evidence to reconstruct the attack timeline and identify the suspect's actions. This investigation demonstrates the importance of Registry analysis, ShellBags, LNK files, and Jump Lists in modern digital forensic investigations.

