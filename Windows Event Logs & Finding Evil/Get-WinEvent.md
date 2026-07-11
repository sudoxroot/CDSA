

# What is `Get-WinEvent`?

`Get-WinEvent` is a **PowerShell cmdlet** used to read Windows Event Logs.

Think of it as:

```text
Windows Event Logs
        │
        ▼
Get-WinEvent
        │
        ▼
Search, Filter, Analyze
```

Instead of opening **Event Viewer** and clicking through thousands of events, you can query them directly from PowerShell.

---

# Why is it useful?

Suppose Windows has

```text
Security Log
50,000 events
```

You don't want to manually search them.

Instead:

```powershell
Get-WinEvent
```

retrieves only the events you need.

---

# Listing Available Logs

```powershell
Get-WinEvent -ListLog *
```

## What does `-ListLog` do?

It **doesn't retrieve events**.

It lists all available event logs.

Example:

```text
System
Security
Application
Windows PowerShell
Microsoft-Windows-Sysmon/Operational
Microsoft-Windows-WinRM/Operational
```

Think of it like:

```text
Event Viewer

├── Security
├── System
├── Application
├── Sysmon
└── WinRM
```

---

## Why use `Select-Object`?

```powershell
Select-Object LogName, RecordCount
```

Instead of displaying everything, it shows only selected columns.

Example:

| Log      | Records |
| -------- | ------- |
| Security | 8968    |
| System   | 1786    |

Much cleaner.

---

# Understanding Log Properties

## LogName

The name of the log.

Example

```text
Security
System
Application
```

---

## RecordCount

Number of events inside.

Example

```text
Security

8968 events
```

---

## IsClassicLog

Indicates whether the log is in the older/classic format.

**True**

Classic Windows logs

```text
Application
System
Security
```

**False**

Modern Windows Event Log channels

```text
Microsoft-Windows-Sysmon/Operational
```

---

## IsEnabled

Whether logging is active.

```text
True
```

Windows is recording events.

```text
False
```

No events are being recorded.

---

## LogMode

Defines what happens when the log becomes full.

### Circular

```text
Newest Event
↓

Oldest event deleted
```

Most common.

---

### Retain

Never overwrite.

Administrator must clear logs manually.

---

### AutoBackup

When full

```text
Current Log

↓

Backup created

↓

New log starts
```

---

## LogType

Indicates the purpose of the log.

### Administrative

Errors

Warnings

Important events

---

### Operational

Normal operational activity.

Most hunting happens here.

---

### Analytical

Very detailed debugging.

---

### Debug

Developer diagnostics.

---

# Listing Providers

```powershell
Get-WinEvent -ListProvider *
```

---

## What is a Provider?

A provider is the **source** that generates events.

Think of it as:

```text
Program

↓

Provider

↓

Event Log
```

Example

```text
Sysmon

↓

Microsoft-Windows-Sysmon

↓

Sysmon Log
```

Another

```text
WinRM

↓

Microsoft-Windows-WinRM

↓

WinRM Log
```

---

# Reading Events

```powershell
Get-WinEvent -LogName System
```

Now you're reading events instead of listing logs.

---

## `-MaxEvents`

```powershell
-MaxEvents 50
```

Only return

```text
50 events
```

instead of thousands.

---

# `Select-Object`

```powershell
Select TimeCreated,ID,ProviderName
```

Shows

- Time
    
- Event ID
    
- Provider
    
- Message
    

instead of hundreds of properties.

---

# Reading WinRM Logs

```powershell
Get-WinEvent -LogName Microsoft-Windows-WinRM/Operational
```

Useful when investigating

- PowerShell Remoting
    
- WinRM abuse
    
- Lateral movement
    

---

# `-Oldest`

Normally

```text
Newest
↓

Older
↓

Oldest
```

Windows returns the newest events first.

Adding

```powershell
-Oldest
```

changes it to

```text
Oldest

↓

Newest
```

Useful when investigating **how an attack started**.

---

# Reading `.evtx` Files

Suppose someone sends you

```text
Security.evtx
```

Instead of importing it into Event Viewer

```powershell
Get-WinEvent -Path C:\Logs\Security.evtx
```

You can query it directly.

Very useful during incident response.

---

# `FilterHashtable`

This is probably the **most commonly used filter**.

Example

```powershell
Get-WinEvent -FilterHashtable @{
    LogName='Microsoft-Windows-Sysmon/Operational'
    ID=1,3
}
```

Meaning

```text
Only search

Sysmon Log

AND

Only Event IDs

1

3
```

---

## Event IDs

Event ID **1**

```text
Process Creation
```

Example

```text
cmd.exe started
```

---

Event ID **3**

```text
Network Connection
```

Example

```text
cmd.exe connected to 8.8.8.8
```

---

Seeing

```text
Event 1

↓

Immediately

↓

Event 3
```

is interesting.

A newly started process immediately connecting to the Internet may indicate malware contacting a **Command and Control (C2)** server.

---

# Filtering by Date

Example

```powershell
StartTime

EndTime
```

instead of

```text
Search all logs
```

Windows searches

```text
Only between

May 28

↓

June 3
```

Much faster.

> **Note:** `EndTime` is **exclusive**, so to include all events on June 2, you set `EndTime` to June 3.

---

# XML Parsing

Events are actually stored internally as XML.

Example

```xml
<Event>
   <EventData>
      <Data Name="ProcessId">1234</Data>
   </EventData>
</Event>
```

PowerShell converts

```powershell
$xml=[xml]$_.ToXml()
```

Now you can access fields like

```powershell
SourceIp

DestinationIp

ProcessGuid

ProcessId
```

instead of parsing raw text.

---

# Why `ProcessGuid` Matters

Every process has

```text
Process ID (PID)
```

Problem:

Windows reuses PIDs.

Example

```text
PID 2000

↓

Process exits

↓

Later another process

↓

PID 2000
```

Confusing.

Instead Sysmon assigns

```text
ProcessGuid

{52ff3419-...}
```

This is unique.

You can correlate:

```text
Process Created

↓

Network Connection

↓

File Created

↓

Registry Changed
```

using the same `ProcessGuid`.

---

# `FilterXml`

Sometimes `FilterHashtable` isn't powerful enough.

Instead

```powershell
FilterXml
```

lets you write XML queries.

Example

Search

```text
Event ID 7

AND

clr.dll

OR

mscoree.dll
```

This detects

```text
.NET Runtime Loading
```

Useful for identifying .NET tools like Seatbelt.

---

# `FilterXPath`

XPath is another query language for XML.

Example

```powershell
Image = reg.exe

AND

CommandLine contains

Sysinternals
```

Much more precise than filtering everything afterward.

---

Another example

```powershell
DestinationIp

=

52.113.194.132
```

Only returns connections to that IP.

---

# Viewing All Properties

```powershell
Select *
```

displays everything.

Example

```text
Image

CommandLine

ParentImage

ParentCommandLine

Hashes

IntegrityLevel

User

ProcessGuid
```

Very useful when you don't know which fields are available.

---

# Filtering by Property

One example from the notes:

```powershell
Where-Object {
    $_.Properties[21].Value -like "*-enc*"
}
```

Let's break it down:

- `$_` → the current event in the pipeline.
    
- `.Properties[21]` → the **22nd field** in a Sysmon Process Create event. For **Event ID 1**, index `21` corresponds to **ParentCommandLine**.
    
- `.Value` → the actual command line text.
    
- `-like "*-enc*"` → searches for the string `-enc` anywhere in that command line.
    

Why `-enc`?

PowerShell supports:

```powershell
powershell.exe -EncodedCommand <Base64>
```

or simply

```powershell
powershell.exe -enc <Base64>
```

Attackers frequently use this to hide malicious scripts. So this query finds processes whose **parent** was launched with an encoded PowerShell command.

---

# CPTS Cheat Sheet 📝

| Command                        | Purpose                                          |
| ------------------------------ | ------------------------------------------------ |
| `Get-WinEvent -ListLog *`      | List all available event logs                    |
| `Get-WinEvent -ListProvider *` | List all event providers                         |
| `Get-WinEvent -LogName System` | Read events from a log                           |
| `-MaxEvents`                   | Limit the number of returned events              |
| `-Oldest`                      | Return oldest events first                       |
| `-Path`                        | Read events from an exported `.evtx` file        |
| `-FilterHashtable`             | Fast filtering by log name, event ID, date, etc. |
| `-FilterXml`                   | Advanced XML-based filtering                     |
| `-FilterXPath`                 | Precise filtering using XPath expressions        |
| `Select-Object`                | Display only selected properties                 |
| `Where-Object`                 | Filter results after retrieval                   |
| `Select *`                     | Show every property in an event                  |

## Exam Tips

- **Event ID 1 (Sysmon)** → Process Creation
    
- **Event ID 3 (Sysmon)** → Network Connection
    
- **Event ID 7 (Sysmon)** → Image/DLL Loaded
    
- Prefer **`-FilterHashtable`** for most investigations because it's simpler and efficient.
    
- Use **`-FilterXml`** or **`-FilterXPath`** when you need to search specific XML fields (e.g., `ImageLoaded`, `DestinationIp`, `CommandLine`).
    
- Use **`ProcessGuid`** instead of **PID** to correlate events across logs, because PIDs can be reused after processes exit.






  

## Question 1

### Utilize the Get-WinEvent cmdlet to traverse all event logs located within the "C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement" directory and determine when the \\*\PRINT share was added. Enter the time of the identified event in the format HH:MM:SS as your answer.

## Solution:

-  It was a quite easy task, I entered the following command in remote desktop
- ```
  Get-WinEvent -FilterHashtable @{Path='C:\Tools\chainsaw\EVTX-ATTACK-SAMPLES\Lateral Movement\*.evtx'; Id='5142'} | Format-List -Property TimeCreated, Message
  ```
- Used event ID 5142 as it was for added shares













