

# Event Tracing for Windows (ETW)

## What is ETW?

**ETW (Event Tracing for Windows)** is Microsoft's high-speed event collection framework built directly into Windows.

Think of it like this:

```text
Applications
Windows Components
Drivers
Antivirus
PowerShell
Sysmon
        │
        ▼
      ETW
(Event Collection Engine)
        │
        ▼
Consumers (Event Viewer, Sysmon, Defender, SIEM, EDR)
```

Instead of every application inventing its own logging system, they can simply send events to ETW.

---

# Why ETW is Important

Normal Windows Security logs only record certain security events.

ETW can monitor much more.

Examples:

- Process creation
    
- Process termination
    
- DLL loading
    
- File creation
    
- Registry changes
    
- Network activity
    
- DNS requests
    
- PowerShell execution
    
- Driver loading
    
- System calls
    

This gives defenders much richer telemetry.

---

# Why SOC Analysts Care

Suppose malware does this:

```text
powershell.exe
        │
Loads malicious DLL
        │
Creates process
        │
Connects to attacker
        │
Edits registry
```

Security Log might only show:

```
4624
User logged in
```

ETW can show:

```
PowerShell command

↓

DLL loaded

↓

Registry modified

↓

Network connection

↓

DNS lookup
```

Much better visibility.

---

# ETW Architecture

Microsoft's ETW follows a **Publisher–Subscriber** model.

Think of it like YouTube.

```text
YouTube Creator

↓

Uploads video

↓

Subscribers receive it
```

ETW works similarly.

```text
Provider

↓

ETW Session

↓

Consumer
```

---

# Components of ETW

---

## 1. Controllers

Controllers manage ETW sessions.

Responsibilities:

- Start tracing
    
- Stop tracing
    
- Enable providers
    
- Disable providers
    

Example:

```cmd
logman.exe
```

Think of the controller as the **manager**.

---

## 2. Providers 

Providers generate events.

Example providers:

```
PowerShell

Kernel

DNS

Registry

Winlogon

SMB

Sysmon
```

Each provider specializes in something.

Example:

```
Microsoft-Windows-PowerShell

↓

PowerShell commands
```

```
Microsoft-Windows-Kernel-Network

↓

Network traffic
```

---

### Types of Providers

You don't need to memorize all four, just know they exist.

| Provider     | Purpose                    |
| ------------ | -------------------------- |
| MOF          | Older format               |
| WPP          | Mainly kernel debugging    |
| Manifest     | Modern XML-based           |
| TraceLogging | Simplified modern provider |

Just remember:

> Providers generate ETW events.

---

## 3. Consumers

Consumers receive ETW events.

Examples:

- Event Viewer
    
- Sysmon
    
- Defender
    
- Elastic Agent
    
- SIEM
    
- EDR
    

Think:

```
Provider

↓

Consumer reads events
```

---

## 4. Channels

Channels organize events.

Like folders.

Example:

```
Operational

Diagnostic

Analytical

Debug
```

Consumers subscribe only to channels they need.

---

## 5. ETL Files

Events are stored inside:

```
*.etl
```

Event Trace Log files.

Useful for:

- DFIR
    
- Malware analysis
    
- Offline investigation
    

---

# ETW Flow

Easy diagram:

```text
PowerShell
Registry
Kernel
DNS
Winlogon

        │

        ▼

   ETW Provider

        │

        ▼

 Trace Session

        │

        ▼

Consumer

(Event Viewer)

(Sysmon)

(Defender)

(Elastic)
```

---

# Interacting with ETW

Windows includes:

```cmd
logman.exe
```

Think of Logman as:

> "Task Manager for ETW"

---

## View Running Sessions

```cmd
logman query -ets
```

Shows:

```
Sysmon Session

Windows Update

Security

Kernel

PowerShell

DNS

...
```

Useful for seeing what Windows is currently tracing.

---

## View Session Details

```cmd
logman query "EventLog-System" -ets
```

Shows:

- Session name
    
- Providers
    
- Buffer size
    
- Log file
    
- Keywords
    
- Event levels
    

---

## List Every Provider

```cmd
logman query providers
```

Outputs hundreds of providers.

Example:

```
PowerShell

Winlogon

Kernel

DNS

OpenSSH

SMB

Registry
```

Windows has over **1,000 built-in providers**.

---

## Search for a Provider

Instead of scrolling:

```cmd
logman query providers | findstr "Winlogon"
```

Returns only:

```
Microsoft-Windows-Winlogon
```

Much faster.

---

## Provider Details

```cmd
logman query providers Microsoft-Windows-Winlogon
```

Shows:

- GUID
    
- Keywords
    
- Event Levels
    
- Processes using it
    

Useful during investigations.

---

# Useful ETW Providers

These are the ones worth remembering.

| Provider                          | Detects                    |
| --------------------------------- | -------------------------- |
| Microsoft-Windows-Kernel-Process  | Process creation/injection |
| Microsoft-Windows-Kernel-File     | File activity              |
| Microsoft-Windows-Kernel-Network  | Network traffic            |
| Microsoft-Windows-PowerShell      | PowerShell commands        |
| Microsoft-Windows-DNS-Client      | DNS queries                |
| Microsoft-Windows-Kernel-Registry | Registry changes           |
| Microsoft-Windows-CodeIntegrity   | Driver/code validation     |
| OpenSSH                           | SSH logins                 |
| WinRM                             | Remote PowerShell          |
| SMBClient / SMBServer             | SMB activity               |
| DotNETRuntime                     | .NET assemblies            |
| TerminalServices                  | RDP sessions               |
| VPN Client                        | VPN connections            |
| Antimalware Service               | Defender activity          |

---

# Real Detection Examples

---

## PowerShell Attack

Provider:

```
Microsoft-Windows-PowerShell
```

Can reveal:

```
powershell.exe

↓

Invoke-WebRequest

↓

Download malware

↓

Execute
```

---

## DNS Tunneling

Provider:

```
Microsoft-Windows-DNS-Client
```

Can detect:

```
aaaaa.example.com

bbbbbb.example.com

cccccc.example.com
```

Large, unusual DNS requests may indicate command-and-control (C2) traffic or data exfiltration.

---

## Registry Persistence

Provider:

```
Kernel Registry
```

Detects:

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

If malware creates a Run key:

```
Registry Provider

↓

Alert
```

---

## RDP Monitoring

Provider:

```
TerminalServices
```

Can detect:

- Remote Desktop logins
    
- Failed RDP attempts
    
- Session creation
    

---

## WinRM

Provider:

```
WinRM
```

Detects:

- Remote PowerShell
    
- Lateral movement
    
- Remote administration
    

---

# Restricted Providers

Some ETW providers are **restricted** because they expose very sensitive information.

Example:

```
Microsoft-Windows-Threat-Intelligence
```

Only privileged processes (typically security software running as **Protected Process Light (PPL)**) can access them.

These providers are valuable because they expose telemetry that helps detect sophisticated attacks, but Windows limits access to protect that data.

---

# GUI Alternatives

Instead of `logman`, you can also use:

- **Performance Monitor (perfmon.exe)** — View and manage ETW sessions graphically.
    
- **EtwExplorer** — Browse provider metadata, events, keywords, and GUIDs.
    

---

# ETW vs Windows Event Logs vs Sysmon

| Feature                   | Windows Event Logs | Sysmon              | ETW               |
| ------------------------- | ------------------ | ------------------- | ----------------- |
| Built into Windows        | ✅                  | ❌                   | ✅                 |
| Authentication events     | ✅                  | Limited             | Through providers |
| Process creation          | Limited (4688)     | ✅ Rich details      | ✅                 |
| DLL loads                 | ❌                  | ✅ (Event ID 7)      | ✅                 |
| Registry monitoring       | Limited            | ✅                   | ✅                 |
| Network connections       | Limited            | ✅ (Event ID 3)      | ✅                 |
| Foundation for many tools | ❌                  | Uses ETW internally | ✅                 |

---

# Exam Notes

## Remember these commands

```cmd
logman query -ets
```

➡️ List running ETW sessions.

---

```cmd
logman query providers
```

➡️ List all available ETW providers.

---

```cmd
logman query providers | findstr "PowerShell"
```

➡️ Search for a specific provider.

---

```cmd
logman query "EventLog-System" -ets
```

➡️ Display details of a trace session.

---

# Memory Trick

Think of ETW as a news network:

```text
Provider
   ↓
Reporter

Controller
   ↓
TV Producer

ETW Session
   ↓
TV Channel

Consumer
   ↓
Viewer

ETL File
   ↓
Recorded Broadcast
```

---

# Key Takeaways

- **ETW is the underlying Windows event tracing framework** that collects telemetry from the operating system, applications, drivers, and security tools.
    
- **Providers generate events**, **controllers manage trace sessions**, and **consumers** (such as Sysmon, Defender, Event Viewer, or an EDR) receive and analyze those events.
    
- **ETL files** store ETW data for offline analysis and forensic investigations.
    
- `logman.exe` is the primary command-line tool for viewing ETW sessions and providers.
    
- Many advanced detections—PowerShell abuse, DNS tunneling, registry persistence, RDP activity, and .NET execution—rely on ETW providers.
    
- **Sysmon complements ETW** by exposing selected ETW telemetry as structured Windows event logs, making it easier for SOC analysts to query and investigate in SIEM platforms.
  










