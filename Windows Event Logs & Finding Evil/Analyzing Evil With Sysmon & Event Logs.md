
# What is Sysmon?

**Sysmon (System Monitor)** is a Microsoft Sysinternals tool that provides **advanced Windows logging**.

Normal Windows logs tell you things like:

```text
User logged in
Process started
Service installed
```

Sysmon adds much deeper visibility:

```text
Which process started?

Which DLL was loaded?

Which IP did it connect to?

Which process accessed LSASS?

Which file was created?
```

Think of it like this:

| Windows Security Logs  | Sysmon                       |
| ---------------------- | ---------------------------- |
| Basic visibility       | Deep visibility              |
| Authentication focused | Endpoint activity focused    |
| Good for auditing      | Excellent for threat hunting |

---

# Sysmon Components

Sysmon consists of:

### 1. Windows Service

Runs continuously.

```text
sysmon.exe
```

Collects events.

---

### 2. Kernel Driver

Captures low-level activity.

Examples:

- Process creation
    
- DLL loading
    
- Registry activity
    
- Process injection
    

---

### 3. Event Log

Stores captured data.

Location:

```text
Applications and Services Logs
    ↓
Microsoft
    ↓
Windows
    ↓
Sysmon
    ↓
Operational
```

---

# Why Sysmon Is Valuable

Consider a normal Windows process creation event:

### Event ID 4688

```text
powershell.exe started
```

Useful.

But Sysmon Event ID 1 gives:

```text
powershell.exe

Parent Process:
explorer.exe

Command Line:
powershell -enc ...

Hashes:
MD5
SHA256
IMPHASH
```

Much richer telemetry.

---

# Common Sysmon Event IDs

These are the ones CPTS students should know well.

| Event ID | Description        |
| -------- | ------------------ |
| 1        | Process Creation   |
| 3        | Network Connection |
| 7        | Image/DLL Load     |
| 8        | CreateRemoteThread |
| 10       | Process Access     |
| 11       | File Creation      |
| 13       | Registry Value Set |
| 22       | DNS Query          |

---

# Sysmon Configuration

Sysmon does not log everything by default.

Instead it uses an XML configuration.

Popular configs:

- [SwiftOnSecurity Sysmon Config](https://github.com/SwiftOnSecurity/sysmon-config?utm_source=chatgpt.com)
    
- [Olaf Hartong Sysmon Modular](https://github.com/olafhartong/sysmon-modular?utm_source=chatgpt.com)
    

These configurations:

- Reduce noise
    
- Focus on attacker techniques
    
- Improve detection quality
    

---

# Detection Example 1 : DLL Hijacking

---

## What is DLL Hijacking?

Windows loads DLLs when programs start.

Example:

```text
calc.exe
    ↓
loads
    ↓
wininet.dll
```

Attackers can sometimes trick Windows into loading:

```text
Malicious WININET.dll
```

instead of the legitimate one.

---

# Sysmon Event ID 7

Event ID:

```text
7
```

Logs:

```text
Image Loaded
```

Example:

```text
Process:
calc.exe

Loaded DLL:
WININET.dll
```

---

# Normal Load

```text
Image:
C:\Windows\System32\calc.exe

DLL:
C:\Windows\System32\WININET.dll

Signed:
True
```

Looks normal.

---

# Malicious Load

```text
Image:
Desktop\calc.exe

DLL:
Desktop\WININET.dll

Signed:
False
```

Huge red flag.

---

# DLL Hijacking IOCs

### IOC #1

Calc.exe outside System32

Normal:

```text
C:\Windows\System32\calc.exe
```

Suspicious:

```text
Desktop\calc.exe
```

---

### IOC #2

WININET.dll outside System32

Normal:

```text
System32\WININET.dll
```

Suspicious:

```text
Desktop\WININET.dll
```

---

### IOC #3

Unsigned DLL

Normal Microsoft DLL:

```text
Signed: True
```

Malicious replacement:

```text
Signed: False
```

---

# Detection Logic

Think:

```text
calc.exe
AND
WININET.dll
AND
Not System32
```

This is very strong evidence of DLL hijacking.

---

# Detection Example 2 : Unmanaged PowerShell / Execute-Assembly

---

## Managed vs Unmanaged Processes

C# programs run inside:

```text
CLR
(Common Language Runtime)
```

Important DLLs:

```text
clr.dll
clrjit.dll
```

---

# Normal Example

PowerShell:

```text
powershell.exe
```

loads:

```text
clr.dll
clrjit.dll
```

Expected.

PowerShell is a .NET application.

---

# Suspicious Example

Suppose:

```text
spoolsv.exe
```

loads:

```text
clr.dll
```

Why?

Print Spooler doesn't normally execute .NET code.

This suggests:

```text
Execute-Assembly
PowerShell Injection
C# Injection
```

---

# IOC

Sysmon Event ID 7:

```text
Image:
spoolsv.exe

ImageLoaded:
clr.dll
```

This should immediately attract attention.

---

# Hunting Rule

Ask:

> Why is a non-.NET process loading CLR components?

Examples:

```text
spoolsv.exe
lsass.exe
winlogon.exe
```

loading

```text
clr.dll
clrjit.dll
```

can indicate malicious code execution.

---

# Detection Example 3 : Credential Dumping

One of the most important Sysmon detections.

---

## What is Credential Dumping?

Attackers want passwords.

Windows stores credential material in:

```text
LSASS.exe
```

(Local Security Authority Subsystem Service)

---

# Mimikatz Example

Common command:

```text
sekurlsa::logonpasswords
```

Goal:

```text
Read LSASS memory
```

to extract:

- Passwords
    
- NTLM hashes
    
- Kerberos tickets
    

---

# Sysmon Event ID 10

Event:

```text
Process Access
```

Logs when one process accesses another process.

---

# Normal Example

```text
AV.exe
    ↓
Accesses
    ↓
lsass.exe
```

Usually legitimate.

---

# Suspicious Example

```text
Downloads\AgentEXE.exe
    ↓
Accesses
    ↓
lsass.exe
```

Very suspicious.

---

# Event ID 10 Fields

### SourceImage

Who initiated access?

Example:

```text
AgentEXE.exe
```

---

### TargetImage

Who was accessed?

Example:

```text
lsass.exe
```

---

### SourceUser

User performing action.

Example:

```text
waldo
```

---

### TargetUser

Usually:

```text
SYSTEM
```

because LSASS runs as SYSTEM.

---

# Credential Dumping IOCs

### IOC #1

Unknown process accessing LSASS.

```text
AgentEXE.exe
→ lsass.exe
```

---

### IOC #2

Process running from odd location.

```text
Downloads\
Desktop\
Temp\
```

---

### IOC #3

SeDebugPrivilege request.

Before dumping LSASS, Mimikatz commonly requests:

```text
SeDebugPrivilege
```

This privilege allows reading memory from other processes.

---

# Quick CPTS Memorization Table

| Sysmon Event ID | What It Detects                    |
| --------------- | ---------------------------------- |
| **1**           | Process creation                   |
| **3**           | Network connections                |
| **7**           | DLL/Image loads                    |
| **8**           | Remote thread creation (injection) |
| **10**          | Process access (LSASS dumping)     |
| **11**          | File creation                      |
| **13**          | Registry modifications             |
| **22**          | DNS queries                        |

---

# Detection Mindset for CPTS

When reviewing Sysmon logs, always ask:

### Event ID 1

> Is this process expected?

---

### Event ID 3

> Is this network connection expected?

---

### Event ID 7

> Is this DLL expected in this process?

---

### Event ID 10

> Why is this process touching LSASS?

---

### Event ID 13

> Why was this registry key modified?

---

### Event ID 22

> Why is this process querying this domain?

---

The biggest CPTS takeaway is that **Sysmon gives context that standard Windows logs often miss**. Many real-world detections for DLL hijacking, process injection, PowerShell abuse, persistence, and credential dumping rely heavily on Sysmon Event IDs **1, 7, and 10**, so make those second nature.

-   
    
    ## Question 1
    
    ### Replicate the DLL hijacking attack described in this section and provide the SHA256 hash of the malicious WININET.dll as your answer. "C:\Tools\Sysmon" and "C:\Tools\Reflective DLLInjection" on the spawned target contain everything you need.
    
    ## Solution:
    - Firstly, I opend sysmon conf file (in notepad) to check the log settings
    - then I looked for ImageLoad section and changed `include` into `exclude`
    - after that, I navigate to DLL injection file and fetch the desired file.exe and file.dll from that folder to desktop and run it (got the hello world pop up)
    - After running that, I went to sysmon logs (in event viewer), I saw the recent logs and corelated things.
    - Got the answerrr
    

    ### Replicate the Unmanaged PowerShell attack described in this section and provide the SHA256 hash of clrjit.dll that spoolsv.exe will load as your answer. "C:\Tools\Sysmon" and "C:\Tools\PSInject" on the spawned target contain everything you need.
    
    ## Solution:
    - I navigated into the PSInject tools:

	C:\Tools\PSInject\
	
	- Then executed the injection via:
	```
	powershell -ep bypass  
	Import-Module .\Invoke-PSInject.ps1  
	Invoke-PSInject -ProcId <PID_of_spoolsv> -PoshCode "<base64_payload>"
	```
	
	- Once injected, I verified the module loads through:
	
	- Event Viewer → Applications and Services Logs → Microsoft → Windows → Sysmon
	
	- Filtering by **Event ID 7**, I located the entry that showed `spoolsv.exe` loading `.NET` DLLs such as:
	
	- `clr.dll`
	- `clrjit.dll`
	
	- To get the hash of the injected JIT compiler DLL, I ran:
	
```
	Get-FileHash "C:\Windows\Microsoft.NET\Framework64\v4.0.30319\clrjit.dll" -Algorithm SHA256
```
    
- ## Question 3
 
    ### Replicate the Credential Dumping attack described in this section and provide the NTLM hash of the Administrator user as your answer. "C:\Tools\Sysmon" and "C:\Tools\Mimikatz" on the spawned target contain everything you need.
    
    ## Solution:
    - I navigated to the C:\Tools\Mimikatz, ran
      ```
      AgentEXE.exe
      ```
	- then 
	```
	  privilege::debug
	```
	- Then executed the logon credential dump:

```
sekurlsa::logonpasswords
```

From the output, I located the **Administrator** block under the **msv** authentication package and extracted the NTLM hash (redacted here).

Sysmon’s **Event ID 10 (ProcessAccess)** validated the access attempt to LSASS during the attack.
	  
	  