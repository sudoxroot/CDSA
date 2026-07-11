
# Detection Example 1: Detecting Strange Parent-Child Relationships

## What is a Parent-Child Relationship?

Every process in Windows is created by another process.

Example:

```
explorer.exe
    │
    ├── chrome.exe
    ├── notepad.exe
    └── cmd.exe
```

- **Parent** = Process that starts another process.
    
- **Child** = Process that gets started.
    

---

## Why is this useful?

Most Windows processes have **predictable parents**.

For example,

```
explorer.exe
      ↓
 notepad.exe
```

This is normal because users usually open Notepad from Explorer.

---

### Another normal example

```
cmd.exe
      ↓
 powershell.exe
```

Normal.

---

## Strange Parent-Child Relationship

Suppose you see

```
calc.exe
      ↓
 cmd.exe
```

This is strange.

Why?

Calculator normally **never launches Command Prompt**.

This could indicate:

- Malware
    
- Exploit
    
- Attacker activity
    

---

## Another Example

Normally,

```
spoolsv.exe
      ↓
 conhost.exe
```

is expected.

`spoolsv.exe` is the **Print Spooler service**.

Sometimes it creates `conhost.exe` because Windows needs a console host.

---

But if you see

```
spoolsv.exe
      ↓
 whoami.exe
```

or

```
spoolsv.exe
      ↓
 cmd.exe
```

that's suspicious.

Print Spooler has **no reason** to execute these programs.

---

# Parent PID Spoofing

Attackers know defenders monitor parent-child relationships.

So they fake them.

This technique is called **Parent PID Spoofing**.

Instead of

```
PowerShell
     ↓
 cmd.exe
```

they make Windows report

```
spoolsv.exe
      ↓
 cmd.exe
```

even though PowerShell actually created it.

---

## Why?

To fool detection systems.

Security software may think

> "Looks like Print Spooler started cmd.exe."

instead of

> "PowerShell launched cmd.exe."

---

## What did the lab do?

PowerShell executed

```
CreateProcessFromParent()
```

using the PID of `spoolsv.exe`.

So Windows reports

```
Parent:
spoolsv.exe

Child:
cmd.exe
```

even though PowerShell started it.

---

# What did Sysmon show?

Sysmon Event ID 1 recorded

```
Parent Process:
spoolsv.exe

Process:
cmd.exe
```

This information is **incorrect** because of spoofing.

Sysmon trusts the parent PID supplied during process creation.

---

# Why ETW Helps

Instead of trusting only the Parent PID,

ETW captures lower-level kernel events.

Specifically,

```
Microsoft-Windows-Kernel-Process
```

provider.

---

Using SilkETW

```
SilkETW.exe
    ↓
Kernel Process Provider
```

the generated JSON showed

```
powershell.exe
        ↓
cmd.exe
```

the **real creator**.

---

## Why?

Kernel ETW records process creation closer to the operating system.

It provides deeper visibility than standard event logs.

---

# Detection Flow

```
Attacker
     │
PowerShell
     │
Parent PID Spoofing
     │
cmd.exe
```

Sysmon sees

```
spoolsv.exe
     ↓
cmd.exe
```

ETW sees

```
powershell.exe
      ↓
cmd.exe
```

ETW wins.

---

# Detection Example 2: Detecting Malicious .NET Assembly Loading

---

## First: Living off the Land (LotL)

Old attackers used tools already installed on Windows.

Examples

```
PowerShell

certutil

wmic

bitsadmin

mshta
```

Advantages:

- Already trusted
    
- Signed by Microsoft
    
- Blend in
    

---

## Defenders Improved

Security tools now detect

- suspicious PowerShell
    
- encoded commands
    
- AMSI
    
- Script Block Logging
    

So attackers adapted.

---

# Bring Your Own Land (BYOL)

Instead of using PowerShell,

attackers bring **their own tools**.

Usually written in

```
C#
```

compiled as

```
.exe

.dll
```

.NET assemblies.

---

Instead of

```
powershell.exe
```

they execute

```
Seatbelt.exe

SharpHound.exe

Rubeus.exe

Certify.exe
```

These are all .NET tools.

---

# Why Attackers Like .NET

Several reasons:

## 1. Already Installed

Every Windows computer already has .NET.

No need to install anything.

---

## 2. CLR Handles Memory

Instead of

```
malloc()

free()

delete
```

CLR automatically manages memory using the **Garbage Collector (GC)**.

Less coding.

Fewer crashes.

---

## 3. Execute Directly in Memory

Normally,

```
Download EXE
↓

Save to Disk
↓

Run
```

Antivirus can scan the file.

---

Instead,

attackers can do

```
Download

↓

Load into Memory

↓

Execute
```

No file touches disk.

Much harder to detect.

---

## 4. Huge Built-in Libraries

.NET includes libraries for:

- HTTP
    
- HTTPS
    
- Encryption
    
- Named Pipes
    
- Registry
    
- File Operations
    
- Networking
    

Attackers don't have to write everything themselves.

---

# Example

One famous tool is

**Seatbelt**

Seatbelt collects system information like

- Privileges
    
- Services
    
- Users
    
- Installed Software
    
- Security Settings
    

Attackers often run it after gaining access.

---

# Execute-Assembly

**Cobalt Strike** can execute

```
Seatbelt.exe
```

entirely in memory.

Meaning

```
Disk
❌

Memory
✅
```

---

# Detecting .NET Assemblies

Whenever a .NET program starts,

Windows loads runtime DLLs.

The important ones are

```
clr.dll

mscoree.dll
```

Think of them as

> ".NET engine started."

---

# Sysmon Event ID 7

Event ID 7

```
Image Loaded
```

records every DLL loaded by a process.

Example

```
Seatbelt.exe

↓

clr.dll

↓

mscoree.dll
```

Sysmon logs both DLL loads.

---

## Problem

Windows loads thousands of DLLs.

Event ID 7 produces huge logs.

Also,

it only tells us

```
clr.dll loaded
```

It does **not** tell us

- which methods ran
    
- what assembly did
    
- internal .NET activity
    

---

# ETW to the Rescue

Instead of using Sysmon,

collect

```
Microsoft-Windows-DotNETRuntime
```

provider.

Now ETW records

- Assembly Name
    
- Namespace
    
- Class
    
- Method Names
    
- JIT Compilation
    
- Loader Events
    

Much richer information.

---

# Why only 0x2038?

Collecting every .NET event would overwhelm the system.

Instead,

SilkETW enables only important keywords.

---

## 1. JitKeyword

JIT = Just-In-Time Compilation.

```
IL Code

↓

Machine Code

↓

Execute
```

Logs methods compiled during execution.

Useful to see **what code actually ran**.

---

## 2. InteropKeyword

Logs interactions between

```
Managed Code (.NET)

↓

Unmanaged Code (Windows API)
```

Example:

```
C#
↓

Win32 API

↓

CreateProcess()
```

Very useful because malware often calls native Windows APIs.

---

## 3. LoaderKeyword

Shows

- Assembly loaded
    
- DLL loaded
    
- Version
    
- Module information
    

Very useful for identifying malicious assemblies like Seatbelt or Rubeus.

---

## 4. NGenKeyword

NGen = Native Image Generator.

Normally,

```
IL

↓

JIT

↓

Machine Code
```

But NGen creates native code beforehand.

```
IL

↓

Native Image

↓

Execute
```

Attackers may use precompiled images to reduce JIT events.

Monitoring NGen helps detect this.

---

# Sysmon vs ETW

|Feature|Sysmon|ETW|
|---|---|---|
|Process creation|✅|✅|
|DLL loading|✅|✅|
|Parent PID spoof detection|❌ Can be fooled|✅ Better visibility|
|.NET method names|❌|✅|
|JIT compilation events|❌|✅|
|Managed ↔ Native interactions|❌|✅|
|Assembly loading details|Limited|Detailed|

---

# Quick Revision 

- **Parent process** = The process that creates another process.
    
- **Child process** = The newly created process.
    
- **Normal parent-child relationships** help detect anomalies.
    
- **Parent PID Spoofing** = Attacker fakes the parent process to evade detection.
    
- **Sysmon Event ID 1** logs process creation but can be deceived by spoofed parent PIDs.
    
- **ETW Microsoft-Windows-Kernel-Process** can reveal the actual creator of a process.
    
- **LotL (Living off the Land)** = Abuse trusted Windows tools like PowerShell.
    
- **BYOL (Bring Your Own Land)** = Bring custom .NET tools (e.g., Seatbelt, Rubeus) instead of using built-in utilities.
    
- **Sysmon Event ID 7** logs DLL loads such as `clr.dll` and `mscoree.dll`, indicating a .NET runtime is in use.
    
- **ETW Microsoft-Windows-DotNETRuntime** provides deep visibility into .NET execution, including loaded assemblies, methods, JIT compilation, interop with native code, and runtime loader events.
    
- **ETW complements Sysmon** by exposing details that traditional event logs cannot, making it particularly valuable for detecting advanced or fileless .NET attacks.

## Question 1

### Replicate executing Seatbelt and SilkETW as described in this section and provide the ManagedInteropMethodName that starts with "G" and ends with "ion" as your answer. "c:\Tools\SilkETW_SilkService_v8\v8" and "C:\Tools\GhostPack Compiled Binaries" on the spawned target contain everything you need.

## Solution:
- Firstly I navigated to GhostPack Compiled Binaries and ran the command on cmd:
- ```
  seatbelt.exe TokenPrivileges
  ```
after that I navigated to the SilkETW_SilkServie_v8/v8 and used the given command in the theory of the section
```
.\SilkETW.exe -t user -pn Microsoft-Windows-DotNetRuntime -uk 0x2038 -ot file -p C:\Users\Administrator\Desktop\etw_trace.json
```
- After some sec, I stopped the running command and headed towards the etw_trace.json, 
- there I search for `ManagedInteropMethodName` and after some search, got the Answer.

