
# Data Sources

Threat hunting begins with collecting useful logs.

Common data sources:

- Windows Event Logs
    
- Sysmon
    
- Linux Syslog
    
- Zeek
    
- JSON Logs
    
- BOTS Dataset (Splunk sample data)
    

In this module, the lab contains **581,073 events**.

Retrieve all logs:

```spl
index="main" earliest=0
```

---

# Step 1 - Identify Available Log Sources

Before hunting, identify available **sourcetypes**.

Query:

```spl
index="main" | stats count by sourcetype
```

Example output:

- WinEventLog:Security
    
- WinEventLog:System
    
- WinEventLog:Application
    
- WinEventLog:Sysmon
    
- linux:syslog
    

**Why?**

- Know what logs are available.
    
- Decide which logs are useful.
    

---

# Step 2 - Focus on Sysmon Logs

Query only Sysmon events:

```spl
index="main" sourcetype="WinEventLog:Sysmon"
```

This reduces unnecessary data and speeds up hunting.

---

# Efficient Searching

## ❌ Bad Search

```spl
index="main" *uniwaldo.local*
```

Problems:

- Searches every field.
    
- Very slow.
    
- High resource usage.
    

---

## ✅ Better Search

```spl
index="main" ComputerName="*uniwaldo.local"
```

Advantages:

- Searches one field only.
    
- Much faster.
    
- Less CPU usage.
    
- Produces fewer false positives.
    

---

# Rule for Threat Hunting ⭐

> **Always search specific fields instead of searching every field.**

---

# View All Sysmon Event IDs

Query:

```spl
index="main" sourcetype="WinEventLog:Sysmon"
| stats count by EventCode
```

This tells you which Sysmon events exist.

---

# Important Sysmon Event IDs

|Event ID|Meaning|Threat Hunting Use|
|---|---|---|
|**1**|Process Creation|Detect suspicious parent-child processes|
|**2**|File Creation Time Changed|Time stomping|
|**3**|Network Connection|Detect C2 traffic|
|**4**|Sysmon Service State|Detect Sysmon being stopped|
|**5**|Process Terminated|Detect killed processes|
|**6**|Driver Loaded|Detect malicious drivers (BYOVD)|
|**7**|Image Loaded|Detect DLL hijacking|
|**8**|CreateRemoteThread|Detect code injection|
|**10**|Process Access|Detect LSASS dumping|
|**11**|File Create|Malware dropped files|
|**12**|Registry Create/Delete|Persistence|
|**13**|Registry Value Set|Registry modifications|
|**15**|FileCreateStreamHash|Mark of the Web|
|**16**|Sysmon Config Changed|Detect Sysmon tampering|
|**17**|Pipe Created|PsExec detection|
|**18**|Pipe Connected|Lateral movement|
|**22**|DNS Query|DNS beaconing|
|**23**|File Delete|Cleanup/Ransomware|
|**25**|Process Tampering|Process Herpaderping|

---

# Hunting Suspicious Parent-Child Processes

List all parent-child relationships:

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
| stats count by ParentImage, Image
```

Thousands of results appear.

Reduce noise by focusing on PowerShell and CMD.

```spl
index="main" sourcetype="WinEventLog:Sysmon" EventCode=1
(Image="*cmd.exe" OR Image="*powershell.exe")
| stats count by ParentImage, Image
```

---

# Suspicious Example

Normal:

```
explorer.exe
   └── notepad.exe
```

Suspicious:

```
notepad.exe
     ↓
powershell.exe
```

This is abnormal because Notepad should never launch PowerShell.

---

# Investigate Further

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=1
(Image="*cmd.exe" OR Image="*powershell.exe")
ParentImage="C:\\Windows\\System32\\notepad.exe"
```

The PowerShell command downloaded:

```
http://10.0.0.229:8080/file.exe
```

Huge red flag.

---

# Investigating an IP Address

Find where an IP appears:

```spl
index="main" 10.0.0.229
| stats count by sourcetype
```

Results:

- Sysmon
    
- Linux Syslog
    

Linux logs reveal:

```
10.0.0.229
=
Linux Machine
```

Meaning:

Windows hosts downloaded malware from a Linux server.

---

# Find All Commands Using That IP

```spl
index="main"
10.0.0.229
sourcetype="WinEventLog:Sysmon"
| stats count by CommandLine
```

Look for:

- Invoke-WebRequest
    
- PsExec
    
- PowerShell downloads
    

These strongly indicate compromise.

---

# Find Infected Hosts

```spl
index="main"
10.0.0.229
sourcetype="WinEventLog:Sysmon"
| stats count by CommandLine, host
```

This identifies infected machines.

---

# Detecting DCSync

Query:

```spl
index="main"
EventCode=4662
Access_Mask=0x100
Account_Name!=*$
```

Why?

Event ID **4662** = Active Directory Object Access

Access Mask:

```
0x100
```

means

```
Control Access
```

used during **DCSync**.

Machine accounts end with:

```
$
```

Example:

```
DC01$
```

Removing machine accounts highlights suspicious user activity.

---

# Important GUID

```
1131f6ad-9c07-11d1-f79f-00c04fc2dcd2
```

Represents:

```
DS-Replication-Get-Changes-All
```

Meaning:

User is attempting to replicate Active Directory secrets.

This confirms a **DCSync attack**.

---

# Detecting LSASS Credential Dumping

Sysmon Event:

```
10
```

(Process Access)

Query:

```spl
index="main"
EventCode=10
lsass
| stats count by SourceImage
```

Look for unusual processes opening LSASS.

Example:

```
notepad.exe
```

should NEVER access

```
lsass.exe
```

Very suspicious.

---

# Important Indicator

Granted Access:

```
0x1FFFFF
```

Very high privileges.

Often associated with:

- Mimikatz
    
- Credential dumping
    
- Memory dumping
    

---

# Understanding CallTrace

Some events show:

```
UNKNOWN
```

inside **CallTrace**.

Meaning:

The API call came from:

- Shellcode
    
- Memory
    
- Injected code
    

instead of

```
DLL on disk
```

This is a powerful detection method.

---

# Building a High-Fidelity Alert

Start with:

```spl
index="main"
CallTrace="*UNKNOWN*"
| stats count by EventCode
```

Only Event ID **10** matched.

---

## Improve Alert Accuracy

### Remove self-access

```spl
| where SourceImage!=TargetImage
```

---

### Ignore .NET JIT

Exclude:

```
Microsoft.NET
```

```
ni.dll
```

```
clr.dll
```

---

### Ignore WOW64

Exclude:

```
wow64
```

---

### Ignore Explorer

Exclude:

```
Explorer.exe
```

Explorer performs many legitimate actions and creates noise.

---

## Final Alert

```spl
index="main"
CallTrace="*UNKNOWN*"
SourceImage!="*Microsoft.NET*"
CallTrace!=*ni.dll*
CallTrace!=*clr.dll*
CallTrace!=*wow64*
SourceImage!="C:\\Windows\\Explorer.EXE"
| where SourceImage!=TargetImage
| stats count by SourceImage, TargetImage, CallTrace
```

This produces a **high-confidence alert** with fewer false positives.

---

# Threat Hunting Methodology

1. Collect logs.
    
2. Identify available sourcetypes.
    
3. Focus on relevant logs (Sysmon).
    
4. Use targeted searches.
    
5. Investigate anomalies.
    
6. Correlate data across hosts.
    
7. Validate with additional evidence.
    
8. Build alerts.
    
9. Reduce false positives.
    
10. Continuously improve detections.
    

---

# Exam Tips ⭐

- Use **field-specific searches** instead of wildcard searches for better performance.
    
- `stats count by sourcetype` shows available log types.
    
- `stats count by EventCode` lists Sysmon events.
    
- **Event ID 1** → Process Creation.
    
- **Event ID 10** → Process Access (LSASS dumping).
    
- **Event ID 11** → File Creation.
    
- **Event ID 22** → DNS Queries.
    
- **Event ID 23** → File Deletion.
    
- **Event ID 4662 + Access_Mask=0x100** → Strong indicator of **DCSync**.
    
- `Account_Name!=*$` filters out machine accounts.
    
- `CallTrace="*UNKNOWN*"` often indicates shellcode or memory injection.
    
- Build alerts by **filtering out known legitimate behavior** (e.g., .NET JIT, WOW64, Explorer) to reduce false positives.
    
- Always **investigate anomalies**, not just individual events—correlate processes, hosts, IPs, and command lines to understand the full attack chain.


-   
    
    ## Question 1
    
    ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through an SPL search against all data the other process that dumped lsass. Enter its name as your answer. Answer format: _.exe
    ## Solution:
    -  From the given data, it was Event Id 10 because one process was triggering another one and it was also damping lSASS, so I build up the following command:
    - ```
      index="main" EventCode=10 lsass SourceImage="*.exe" SourceImage!="Sysmon.exe" SourceImage!="Sysmon64.exe" SourceImage!="wininit.exe"
| stats count by SourceImage
      ```
    - There was 8 sort of .exe files but I chose rundll32.exe as attackers frequently abuse `rundll32.exe` because it is a trusted, native Windows binary. It is a favored "Living off the Land" (LotL) tool used to call legitimate Windows libraries (like `comsvcs.dll`) to trigger a `MiniDump`
    
- ## Question 2
    
    
    ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through SPL searches against all data the method through which the other process dumped lsass. Enter the misused DLL's name as your answer. Answer format: _.dll
    - Firstly, I hit upon an idea to check the malicious rundll32.exe, so I search for its PID
    - ```
      index="main" sourcetype="WinEventLog:Sysmon" EventCode=10 TargetImage="*lsass.exe" SourceImage="*rundll32.exe" | table SourceProcessId
      ```
    - We got to know the starting point of the dll file so I search for event id 1 for the events I got, there was 8 different PIDs so I Started checking one by one
    ```
    index="main" sourcetype="WinEventLog:Sysmon" EventCode=1 ProcessId=1624
    ```
    - during checking the first event, I checked the commandline which got the name of a suspicious dll file, I put that name and got the answer
    
- ## Question 3
    
    
    ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through an SPL search against all data any suspicious loads of clr.dll that could indicate a C# injection/execute-assembly attack. Then, again through SPL searches, find if any of the suspicious processes that were returned in the first place were used to temporarily execute code. Enter its name as your answer. Answer format: _.exe
    - this question is divided into 2 parts, first was the indication at C# injection, so I make the following query
    - ```
      index="main" sourcetype="WinEventLog:Sysmon" EventCode=7 ImageLoaded="*clr.dll" | where Image!="C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe" AND Image!="C:\\Windows\\System32\\wsmprovhost.exe" AND Image!="C:\\Windows\\System32\\csc.exe" | stats count by Image | sort Image
      ```
    - It filterout most of the suspicious files, and help me in such a way that I didn't have to make another query for the second file,
    - there I saw `rundll32.exe`, whose only job was to **temporarily execute code** from a DLL.
    
- ## Question 4
    
    ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through SPL searches against all data the two IP addresses of the C2 callback server. Answer format: 10.0.0.1XX and 10.0.0.XX
    - as it's obvious that network connection is created for C2, so I used the event id 3, moreover after suspecting the images of event id 3, I got specific images which are involved in c2 communication.
    - ```
      index="main" sourcetype="WinEventLog:Sysmon" EventCode=3 (Image="*rundll32.exe" OR Image="*cmd.exe" OR Image="*randomfile.exe" OR Image="*SharpHound.exe") | table _time, Image, DestinationIp, DestinationPort
      ```
    - After observing the pattern of the IP addresses I got to know the exact answer.
    
- ## Question 5
    

    ### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through SPL searches against all data the port that one of the two C2 callback server IPs used to connect to one of the compromised machines. Enter it as your answer.
    - Observing the data from the above port, I saw port 3389 (used for RDP connection )
    - the attacker wasn't just sending commands, they were opening a full-blown Remote Desktop session to take interactive control. The "port that was used" to define the attack was `3389`.






