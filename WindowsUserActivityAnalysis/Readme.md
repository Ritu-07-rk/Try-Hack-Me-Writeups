# Windows Forensics — "36 Hours of Rampage" Writeup

## Scenario

James, an HR employee at Cybertees Pvt Ltd, returned to work on Monday to find files missing and unfamiliar tools installed on his workstation. CCTV footage places a soon-to-resign employee, Johny, at James's machine over the weekend — allegedly to steal sensitive documents before joining a competitor. Johny is also believed to have deleted files and run anti-forensic tools to cover his tracks.

**Goal:** reconstruct Johny's 36-hour window of activity — files accessed, tools executed, and traces of anti-forensic behavior — using native Windows artifacts and registry forensics.

---

## Task 3 — Revisiting the Registry

The Windows Registry is a hierarchical database of OS, application, and user configuration data, physically stored as hive files under `%SystemRoot%\System32\config`.

| Hive File | Mapped Key Path | Purpose |
|---|---|---|
| SAM | `HKLM\SAM` | User accounts and local security policy |
| SECURITY | `HKLM\SECURITY` | Authentication and permissions data |
| SYSTEM | `HKLM\SYSTEM` | Hardware, drivers, startup config |
| SOFTWARE | `HKLM\SOFTWARE` | Installed software and system-wide settings |
| DEFAULT | `HKU\.DEFAULT` | Template for new user profiles |
| NTUSER.DAT | `HKCU` | Per-user preferences and settings |
| USRCLASS.DAT | `HKCU\Software\Classes` | Per-user shell/class configuration |

**Dirty hives & transaction logs:** if a hive isn't cleanly closed (e.g. abrupt shutdown), it's marked "dirty." Windows uses transaction logs (`SYSTEM.LOG1`, `SYSTEM.LOG2`, …) stored alongside the hives to roll back or replay changes and keep the registry consistent — useful forensically for spotting recent, uncommitted activity.

### Questions & Answers

| # | Question | Answer |
|---|---|---|
| 1 | Which hive stores information about installed software? | `SOFTWARE` |
| 2 | Current size of the SAM hive in the lab (KB)? | `128` |

---

## Task 4 — Performing Registry Forensics

Several `NTUSER.DAT` keys under `HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\` reveal user-driven activity in File Explorer and the Run dialog:

| Artifact | Registry Path | What it Shows |
|---|---|---|
| **TypedPaths** | `...\Explorer\TypedPaths` | Paths manually typed into the Explorer address bar / Run dialog |
| **WordWheelQuery** | `...\Explorer\WordWheelQuery` | Terms searched via Explorer's search box |
| **RecentDocs** | `...\Explorer\RecentDocs` | Recently opened documents |
| **ComDlg32 → LastVisitedMRU** | `...\Explorer\ComDlg32\LastVisitedMRU` | Recently visited folders (via Open/Save dialogs) |
| **ComDlg32 → OpenSavePidlMRU** | `...\Explorer\ComDlg32\OpenSavePidlMRU` | Recently opened/saved files and their locations |
| **UserAssist** | `...\Explorer\UserAssist` | GUI program execution counts and last-run timestamps |
| **RunMRU** | `...\Explorer\RunMRU` | Commands recently run via the Run dialog |

Two notable **UserAssist** GUIDs:
- `{CEBFF5CD-ACE2-4F4F-9178-9926F41749EA}` — Explorer file/folder interactions
- `{F4E57C4B-2036-45F0-A9AB-443BCFE33D9F}` — shortcut/`.LNK`-based program execution

### Questions & Answers

| # | Question | Answer |
|---|---|---|
| 1 | Suspicious path typed in Explorer pointing to a `tmp` directory on C: — full path? | `C:\system\home\tmp` |
| 2 | Latest term searched in WordWheelQuery? | `wipe` |
| 3 | Where was the last text file saved by the suspect? | `C:\system\home\tmp\code.txt` |
| 4 | Suspicious keylogging tool run 5 times from the Hacking-tools folder? | `keylogger.exe` |
| 5 | Disk wiping utility executed on the host? | `DiskWipe.exe` |

**Takeaway:** UserAssist execution counts directly tie Johny to a keylogger and a disk-wiping tool — strong evidence of both data theft and anti-forensic intent.

---

## Task 5 — ShellBags

ShellBags track how a user browsed folders (view settings, sort order, window size) — and critically, they can persist **even after the folder itself has been deleted**, making them resilient to basic cleanup attempts.

**Stored in:**
- `%USERPROFILE%\NTUSER.dat`
- `%USERPROFILE%\AppData\Local\Microsoft\Windows\UsrClass.dat`

**What ShellBags reveal:**
- Folder view settings (icons/list/details)
- Full paths of accessed directories, including network shares and removable media
- Creation/access/modification timestamps
- Traces of deleted folders
- Evidence of external drive and network share access

### Questions & Answers

| # | Question | Answer |
|---|---|---|
| 1 | IP address of the network share where the user accessed three folders? | `10.10.17.228` |
| 2 | Name of the second sub-folder inside the Documents folder on the network share? | `secret-doc` |

---

## Task 6 — LNK Files & Jump Lists

### LNK (Shortcut) Files
Windows auto-generates a `.lnk` file whenever a user opens a file or application. These shortcuts record the accessed file's name, timestamp, target path, source (local disk vs. network share), and file size.

**Default locations:**
- `%userprofile%\AppData\Roaming\Microsoft\Windows\Recent`
- `%userprofile%\Recent`

**Tool used:** `LECmd.exe` (Eric Zimmerman's LNK parser)

```
LECmd.exe -f C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Recent\code.lnk
```

### Jump Lists
Jump Lists track frequently used files, apps, and sites, surfaced via the Taskbar/Start Menu. They come in two flavors:
- **AutomaticDestinations** — auto-populated per application
- **CustomDestinations** — manually added entries by an app

**Location:**
- `%APPDATA%\Microsoft\Windows\Recent\AutomaticDestinations`
- `%APPDATA%\Microsoft\Windows\Recent\CustomDestinations`

Combined with LNK analysis, Jump Lists exposed which sensitive files (including PDFs) Johny opened, from where, how often, and when.

### Questions & Answers

| # | Question | Answer |
|---|---|---|
| 1 | Full path of the `tmp` directory where `code.txt` was accessed? | `C:\system\home\tmp\code.txt` |
| 2 | URL accessed via Internet Explorer? | `http://10.10.17.228/` |
| 3 | Timestamp Johny accessed "How to Hack.pdf"? | `2024-03-04 12:28:26` |

---

## Investigation Summary

Piecing the artifacts together, Johny's activity over the 36-hour window included:

1. **Reconnaissance** — searching for and browsing sensitive folders via Explorer (TypedPaths, WordWheelQuery, ShellBags), including a network share at `10.10.17.228` containing a `secret-doc` folder.
2. **Data staging** — working out of a local `C:\system\home\tmp` directory, saving/editing `code.txt`.
3. **Malicious tooling** — executing `keylogger.exe` (5 times) from a `Hacking-tools` folder, per UserAssist.
4. **Research/intent** — opening "How to Hack.pdf," and accessing an external URL over HTTP.
5. **Anti-forensics** — running `DiskWipe.exe` to attempt to destroy evidence, though registry, ShellBag, LNK, and Jump List artifacts survived and reconstructed the timeline regardless.

**Key forensic lesson:** even deliberate cleanup (deleting files, wiping disks) leaves a footprint across multiple redundant Windows artifacts — registry MRU keys, ShellBags, LNK files, and Jump Lists — because each is generated and stored independently by different OS subsystems.

---

*Tools referenced: LECmd.exe (Eric Zimmerman's EZ Tools suite).*
