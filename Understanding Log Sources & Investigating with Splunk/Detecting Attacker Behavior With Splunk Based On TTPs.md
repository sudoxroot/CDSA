
# Two Detection Approaches

## 1. TTP-Based Detection (Known Threats)

Detect behavior that matches known attacker techniques.

Think:

> **"I know attackers do this. Let me search for it."**

Examples:

- PowerShell downloads
    
- PsExec
    
- DCSync
    
- Reconnaissance
    
- LSASS dumping
    

**Pros**

- High confidence
    
- Easy to explain
    
- Low false positives (if tuned)
    

**Cons**

- Misses new attack techniques.
    

---

## 2. Anomaly-Based Detection

Detect activity that is **unusual** compared to normal behavior.

Think:

> **"I don't know what the attack is, but this behavior is abnormal."**

Examples:

- Notepad launching PowerShell
    
- Downloads from unusual folders
    
- Rare processes
    
- Connections to uncommon ports
    

**Pros**

- Finds unknown attacks.
    
- Detects zero-days.
    

**Cons**

- More false positives.
    
- Requires tuning.
    

---

# Key Principle

A good SOC combines **both** approaches.

Known attacks → TTP Detection

Unknown attacks → Anomaly Detection

---

# Before Creating Detections

Always understand:

- Your environment
    
- Normal activity
    
- Available logs
    
- Expected behavior
    

Otherwise you'll create many false positives.

---

# Detection 1 - Windows Reconnaissance Commands

Attackers often use built-in Windows tools.

Examples:

- ipconfig
    
- whoami
    
- net
    
- netstat
    
- hostname
    
- tasklist
    
- nbtstat
    

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=1
Image=*\\ipconfig.exe
OR Image=*\\net.exe
OR Image=*\\whoami.exe
OR Image=*\\netstat.exe
OR Image=*\\nbtstat.exe
OR Image=*\\hostname.exe
OR Image=*\\tasklist.exe
| stats count by Image, CommandLine
| sort - count
```

### Why?

Attackers use these commands to:

- Discover users
    
- Discover network
    
- Discover computers
    
- Discover privileges
    

MITRE:

- Discovery
    

---

# Detection 2 - GitHub Payload Downloads

Attackers frequently host malware on:

```text
githubusercontent.com
```

Why?

Many companies whitelist GitHub.

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=22
QueryName="*github*"
| stats count by Image, QueryName
```

### Detects

- PowerShell downloading malware
    
- Scripts hosted on GitHub
    
- Payload delivery
    

MITRE:

Ingress Tool Transfer

---

# Detection 3 - PsExec Usage

PsExec is a legitimate Microsoft Sysinternals tool.

Administrators use it for:

- Remote execution
    
- Remote administration
    

Attackers use it for:

- Lateral movement
    
- Service execution
    
- SMB execution
    

MITRE:

- T1021.002
    
- T1569.002
    
- T1570
    

---

## PsExec Detection Using Event ID 13

Event ID:

```text
13
```

Registry Value Set

Query watches:

```text
HKLM\System\CurrentControlSet\Services\*\ImagePath
```

Reason:

PsExec creates temporary Windows services.

---

## PsExec Detection Using Event ID 11

Query:

```spl
index="main"
EventCode=11
Image=System
| stats count by TargetFilename
```

Detects:

Creation of PsExec service executable.

---

## PsExec Detection Using Event ID 18

Event ID:

```text
18
```

Named Pipe Connected

Query:

```spl
index="main"
EventCode=18
Image=System
| stats count by PipeName
```

PsExec creates named pipes.

Very useful detection.

---

# Detection 4 - Archive File Creation

Attackers often compress:

- Stolen data
    
- Malware
    
- Tools
    

Common extensions:

- .zip
    
- .rar
    
- .7z
    

Query:

```spl
index="main"
EventCode=11
(TargetFilename="*.zip"
OR TargetFilename="*.rar"
OR TargetFilename="*.7z")
| stats count by ComputerName, User, TargetFilename
| sort - count
```

Possible Indicators

- Data exfiltration
    
- Tool transfer
    
- Collection stage
    

---

# Detection 5 - PowerShell Downloads

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=11
Image="*powershell.exe*"
| stats count by Image, TargetFilename
```

Detects:

PowerShell writing downloaded files.

Very common malware behavior.

---

# Detection 6 - Microsoft Edge Downloads

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=11
Image="*msedge.exe"
TargetFilename="*Zone.Identifier"
```

## What is Zone.Identifier?

Windows creates:

```text
Zone.Identifier
```

Alternate Data Stream (ADS)

It indicates:

> This file came from the Internet.

Useful for identifying downloaded files.

---

# Detection 7 - Executables Running from Downloads Folder

Query:

```spl
index="main"
EventCode=1
| regex Image="C:\\\\Users\\\\.*\\\\Downloads\\\\.*"
| stats count by Image
```

Why suspicious?

Users normally execute software from:

- Program Files
    
- Windows
    

Not:

```text
Downloads
```

Common malware examples:

- SharpHound
    
- PsExec64
    
- randomfile.exe
    

---

# Detection 8 - EXE/DLL Created Outside Windows Folder

Query:

```spl
index="main"
EventCode=11
(TargetFilename="*.exe"
OR TargetFilename="*.dll")
TargetFilename!="*\\windows\\*"
| stats count by User, TargetFilename
```

Detects:

- Malware dropped into user folders
    
- DLL side-loading
    
- Payload creation
    

---

# Detection 9 - Misspelled Legitimate Files

Attackers often imitate legitimate binaries.

Example:

Legitimate

```text
PSEXESVC.exe
```

Malicious

```text
PSEXEsvc.exe
PsExevc.exe
PSEXEE.exe
```

Query searches for:

```text
psexe*.exe
```

while excluding legitimate names.

Purpose:

Detect masquerading.

MITRE:

Masquerading

---

# Detection 10 - Non-Standard Ports

Normal ports:

|Port|Protocol|
|---|---|
|80|HTTP|
|443|HTTPS|
|21|FTP|
|22|SSH|

Anything else may deserve investigation.

Query:

```spl
index="main"
EventCode=3
NOT (
DestinationPort=80
OR DestinationPort=443
OR DestinationPort=22
OR DestinationPort=21
)
| stats count by SourceIp, DestinationIp, DestinationPort
| sort - count
```

Detects:

- Reverse shells
    
- C2 traffic
    
- Malware communications
    
- Custom listeners
    

---

# Important Detection Concepts

## EventCode 1

Process Creation

Best for:

- Parent-child analysis
    
- Recon commands
    
- Suspicious execution
    

---

## EventCode 3

Network Connections

Best for:

- C2 traffic
    
- Reverse shells
    
- Non-standard ports
    

---

## EventCode 11

File Creation

Best for:

- Malware downloads
    
- Payload creation
    
- Archive files
    
- Executables
    

---

## EventCode 13

Registry Value Set

Best for:

- Service creation
    
- Persistence
    
- PsExec
    

---

## EventCode 18

Named Pipe Connection

Best for:

- PsExec
    
- SMB activity
    
- Lateral movement
    

---

## EventCode 22

DNS Query

Best for:

- GitHub payloads
    
- C2 domains
    
- DNS tunneling
    

---

# Threat Hunting Mindset

Instead of asking:

> "Is this malware?"

Ask:

- Is this normal?
    
- Who executed it?
    
- Where was it executed?
    
- Why did it happen?
    
- Does it match attacker TTPs?
    
- Does it differ from normal behavior?
    

---

# Detection Workflow

```
Collect Logs
      ↓
Know Your Environment
      ↓
Know Attacker TTPs
      ↓
Write SPL Queries
      ↓
Investigate Results
      ↓
Reduce False Positives
      ↓
Create Reliable Alerts
      ↓
Continuously Improve
```

---

# Exam Tips ⭐

- **TTP-based detection** searches for **known attacker behaviors**.
    
- **Anomaly-based detection** searches for **unusual behavior** that deviates from the norm.
    
- Use **Sysmon Event ID 1** for process creation and reconnaissance commands (`ipconfig`, `whoami`, `net`, etc.).
    
- **Event ID 22** is useful for detecting suspicious DNS queries, including downloads from `githubusercontent.com`.
    
- **PsExec** can be detected using multiple Sysmon events:
    
    - **13** → Registry service creation (`ImagePath`)
        
    - **11** → Service executable file creation
        
    - **18** → Named pipe connections
        
- **Event ID 11** is valuable for detecting:
    
    - PowerShell downloads
        
    - Archive creation (`.zip`, `.rar`, `.7z`)
        
    - EXE/DLL creation outside the Windows directory
        
- `Zone.Identifier` indicates a file was downloaded from the Internet.
    
- Executables running from a user's **Downloads** folder are often suspicious.
    
- Misspelled legitimate binaries (e.g., variations of `PSEXESVC.exe`) are a common **masquerading** technique.
    
- Connections to **non-standard ports** may indicate C2 traffic, reverse shells, or malware communications.
    
- The best detections combine **knowledge of attacker TTPs**, **understanding of normal environment behavior**, and **continuous tuning** to minimize false positives.

  

# Question 1

### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through SPL searches against all data the password utilized during the PsExec activity. Enter it as your answer.

## Solution:
- For searching the password, we must stick to the event id 1 which is created when a process is created, password can be detected by various keywords like `-p` etc or by searching for the psexec in commandline
- ```
  index="main" (CommandLine="*-p*" AND CommandLine=*psexec*) EventCode=1
| table CommandLine , SourceImage
  ```
- the password was so obvious in the result.



