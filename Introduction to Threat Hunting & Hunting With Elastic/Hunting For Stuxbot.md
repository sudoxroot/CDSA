
# Stuxbot Threat Intelligence Report

---

# Threat Summary

| Item             | Value                  |
| ---------------- | ---------------------- |
| Malware          | Stuxbot                |
| Risk             | **Critical**           |
| Target           | Windows users          |
| Goal             | Espionage              |
| Initial Access   | Opportunistic phishing |
| Persistence      | EXE dropped to disk    |
| Lateral Movement | PsExec, WinRM          |

---

# Attacker Profile

The attackers:

- Send phishing emails
    
- Pretend to send invoices
    
- Install a Remote Access Trojan (RAT)
    
- Move laterally
    
- Escalate privileges
    

Unlike ransomware,

They **don't encrypt files**.

Instead they:

- Spy
    
- Collect information
    
- Expand access
    

---

# Initial Infection Chain

This is the most important diagram.

```text
Phishing Email
      │
      ▼
invoice.one
(OneNote file)
      │
      ▼
invoice.bat
(Batch file)
      │
      ▼
PowerShell
(downloads payload)
      │
      ▼
default.exe
(RAT)
      │
      ▼
Persistence
      │
      ▼
Password spraying
      │
      ▼
SharpHound
      │
      ▼
PsExec
      │
      ▼
Compromise PKI Server
```

**Remember this sequence**. It's the story you're reconstructing from the logs.

---

# Phishing Email

Example

```
Invoice #76

Click here to view invoice
```

The link downloads

```
invoice.one
```

---

# OneNote File

Looks harmless.

Actually contains

```
Hidden Button
```

Clicking it runs

```
invoice.bat
```

---

# Batch File

Purpose

Run PowerShell.

Think of it as a launcher.

```
invoice.one

↓

invoice.bat

↓

PowerShell
```

---

# PowerShell

PowerShell downloads the real malware from Pastebin.

Example

```
Pastebin

↓

PowerShell

↓

Execute in memory
```

This is called a **stager** because it downloads the next stage.

---

# RAT

The downloaded RAT can:

- Remote shell
    
- Screen capture
    
- Run commands
    
- Credential dumping
    
- Execute Mimikatz
    

Think

```
Victim PC

↓

Full Remote Control
```

---

# Persistence

Attackers drop

```
default.exe
```

onto disk.

That executable survives reboot.

---

# Lateral Movement

Attackers later move using

- PsExec
    
- Windows Remote Management
    

Exactly as stated in the threat report.

---

# Indicators of Compromise (IOCs)

## Network

Command & Control IPs

```
91.90.213.14

103.248.70.64

141.98.6.59
```

---

## URLs

OneNote

```
invoice.one
```

PowerShell staging

```
Pastebin URLs
```

---

## File Hashes

Three SHA256 hashes identify the malware.

These can be searched directly in Sysmon.

---

# Available Logs

You have multiple log sources.

| Log Source      | Purpose                        |
| --------------- | ------------------------------ |
| Windows Logs    | Authentication, events         |
| Sysmon          | Processes, files, network, DNS |
| PowerShell Logs | PowerShell activity            |
| Zeek            | DNS, HTTP, network traffic     |

Think of it like this:

```
Windows
    │
Sysmon
    │
PowerShell
    │
Zeek

↓

Complete Picture
```

---

# Threat Hunting Walkthrough

The investigation follows the attack timeline.

---

## Step 1

Search

```
invoice.one
```

Query

```kql
event.code:15 AND file.name:*invoice.one
```

Why?

Sysmon Event ID **15** logs **browser download streams**, showing that the OneNote file was downloaded.

Result:

- Bob downloaded `invoice.one`
    
- Browser = Microsoft Edge
    
- Host = WS001
    

---

## Step 2

Confirm file creation.

Query

```kql
event.code:11
```

Why?

Event ID **11** confirms the file was written to disk.

Now we know:

```
Download

↓

Saved to disk
```

---

## Step 3

Find the workstation IP.

Host

```
WS001
```

Network IP

```
192.168.28.130
```

This IP is used for the next searches.

---

## Step 4

Look in Zeek.

Question

"What websites did WS001 visit?"

Query

```kql
source.ip:192.168.28.130
```

Result

```
mail.google.com

↓

file.io

↓

SmartScreen
```

Interpretation:

1. User opened Gmail.
    
2. Downloaded the file from a hosting site.
    
3. Microsoft Defender SmartScreen scanned it.
    

That validates the phishing story.

---

## Step 5

Did the user open the file?

Search

```kql
process.command_line:*invoice.one*
```

Result

OneNote opened

```
invoice.one
```

Only **6 seconds** after download.

That strongly suggests the user executed it.

---

## Step 6

What did OneNote launch?

Query

```kql
process.parent.name:"ONENOTE.EXE"
```

Result

```
cmd.exe

↓

invoice.bat
```

Now we know

```
OneNote

↓

Batch file
```

---

## Step 7

What did the batch file launch?

Query

```kql
process.parent.command_line:*invoice.bat*
```

Result

```
PowerShell
```

PowerShell downloaded content from

```
Pastebin
```

Exactly matching the threat report.

---

## Step 8

Investigate PowerShell

Search by

```
PID
```

Example

```kql
process.pid:9944
```

Now you see everything PowerShell did.

Observed activities include:

- Downloading the payload
    
- Dropping `default.exe`
    
- DNS lookups
    
- Network connections
    
- Password-spraying script
    

This single PID tells the malware's story.

---

## Step 9

Discover the C2

PowerShell resolved

```
ngrok.io
```

Then connected to

```
18.158.249.75
```

Later,

DNS changed

↓

New IP

```
3.125.102.39
```

This illustrates why **IP-based detection is weaker than behavior-based detection**—the infrastructure changed, but the malware's behavior remained the same.

---

## Step 10

Investigate `default.exe`

Search

```kql
process.name:"default.exe"
```

Findings

- Executed
    
- Contacted C2
    
- Uploaded files
    
- Dropped additional malware
    

Files observed

```
svchost.exe

SharpHound.exe

payload.exe
```

---

## Step 11

Investigate SharpHound

Search

```kql
process.name:"SharpHound.exe"
```

Result

Executed twice.

Purpose

Map Active Directory and identify privilege escalation paths.

This is evidence of **Active Directory reconnaissance** after the initial compromise.

---

## Step 12

Search by malware hash

Query

```kql
process.hash.sha256:
018D37CBD3878258...
```

Result

Found on

- WS001
    
- PKI
    

Meaning

The malware spread beyond Bob's workstation.

---

## Step 13

How did it reach PKI?

Look at parent process.

Parent

```
PSEXESVC.exe
```

This is the service created by **PsExec**, confirming the lateral movement method described in the threat report.

```
WS001

↓

PsExec

↓

PKI
```

---

## Step 14

Compromised Account

User

```
svc-sql1
```

appears on the PKI server.

That service account was compromised and used to execute the malware.

---

## Step 15

How was `svc-sql1` compromised?

Look at authentication logs.

Query

```kql
(event.code:4624 OR event.code:4625)
```

Results

```
Administrator

↓

Failed logons

↓

svc-sql1

↓

Successful logons
```

Interpretation

- Attackers tried to guess the **Administrator** password.
    
- They failed.
    
- They later obtained or guessed the **svc-sql1** credentials and used them successfully.
    

---

# Complete Attack Timeline

```text
Phishing Email
        │
        ▼
invoice.one downloaded
(Sysmon 15)
        │
        ▼
File written to disk
(Sysmon 11)
        │
        ▼
OneNote opened
(Sysmon 1)
        │
        ▼
invoice.bat executed
        │
        ▼
PowerShell launched
        │
        ▼
Downloaded payload from Pastebin
        │
        ▼
Dropped default.exe
        │
        ▼
Connected to ngrok C2
        │
        ▼
Dropped SharpHound
        │
        ▼
Mapped Active Directory
        │
        ▼
Password spraying
        │
        ▼
Compromised svc-sql1
        │
        ▼
Used PsExec
        │
        ▼
Compromised PKI server
```

# Exam Takeaways

- Build your investigation **chronologically**. Start with the initial access hypothesis and follow the evidence.
    
- Correlate multiple log sources:
    
    - **Sysmon** → processes, files, DNS, network.
        
    - **Zeek** → DNS and network confirmation.
        
    - **Windows Security Logs** → authentication events (`4624` successful logon, `4625` failed logon).
        
- Validate threat intelligence by matching **IOCs** (hashes, domains, IPs) **and** **TTPs** (PowerShell download, `PsExec`, `SharpHound`, password spraying).
    
- A single IOC rarely proves compromise. Multiple correlated events across different log sources provide strong evidence of an intrusion.

-   
    
    ## Question 1
    
    ### Navigate to http://[Target IP]:5601 and follow along as we hunt for Stuxbot. In the part where default.exe is under investigation, a VBS file is mentioned. Enter its full name as your answer, including the extension.
    ## Solution:
    - from the given info, the general assumption of query can be:
    - ```
      process.name:"default.exe" AND file.name"*.vbs"
      ```
    - There was only one hit and I got the answerrrrrr.
    
- ## Question 2
    
    ### Stuxbot uploaded and executed mimikatz. Provide the process arguments (what is after .\mimikatz.exe, ...) as your answer.
    ## Solution:
    - there was given that something after .\mimikatz.exe would occur, so I used the following query:
    - ```
      ".\mimikatz"
      ```
    - the place was a bit successful but there was 7 hits, then I start using filter and find out process.args 
    - put the filter and got the answer
    
- ## Question 3
    
    ### Some PowerShell code has been loaded into memory that scans/targets network shares. Leverage the available PowerShell logs to identify from which popular hacking tool this code derives. Answer format (one word): P____V___
    ## Solution:
    - This seemed to be quite a simple question:
    - ```
      powershell.file.script_block_text : * AND powershell.file.script_block_text : "IEX"
      ```
- Firstly I ran powershell.file.script_block_text but I came through about a tone of events.
- then a though hit upon my mind that mostly executed script is used with invoke expression, I used the rest of the piece of command
- After running it I reduced the noise but the still the answer was vague.
- after scrutinizing the details, I saw base64 encoding, I copied the encoded lines and decode it
- Faahhhhh, Got the answer
- 

























