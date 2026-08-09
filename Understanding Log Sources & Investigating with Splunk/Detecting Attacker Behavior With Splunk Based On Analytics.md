

# Module Goal

Unlike **TTP-based detection**, analytics-based detection focuses on **finding abnormal behavior** using statistics.

Instead of asking:

> **"Does this match a known attack?"**

we ask:

> **"Is this behavior unusual?"**

This helps detect:

- Unknown attacks
    
- Zero-day malware
    
- Insider threats
    
- New attacker techniques
    

---

# Detection Philosophy

## TTP Detection

Looks for:

✔ Known attacker behavior

Example:

- PsExec
    
- PowerShell download
    
- DCSync
    

---

## Analytics Detection

Looks for:

✔ Unusual behavior

Example:

- Process suddenly making hundreds of connections
    
- Extremely long command lines
    
- Process loading many DLLs
    
- Same process repeatedly executing
    

---

# streamstats Command

One of the most powerful Splunk commands.

Used to calculate statistics over time.

Example:

- Rolling average
    
- Rolling standard deviation
    
- Running totals
    

Think of it as:

> "Compare current activity with past activity."

---

# Detecting Abnormal Network Connections

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=3
| bin _time span=1h
| stats count as NetworkConnections by _time Image
| streamstats time_window=24h avg(NetworkConnections) as avg stdev(NetworkConnections) as stdev by Image
| eval isOutlier=if(NetworkConnections>(avg+(0.5*stdev)),1,0)
| search isOutlier=1
```

---

## Query Breakdown

### EventCode=3

Network Connection

---

### bin

```spl
| bin _time span=1h
```

Groups events into **1-hour buckets**.

Example:

Instead of:

```
10:01
10:02
10:03
```

It becomes

```
10:00–10:59
```

---

### stats

```spl
stats count as NetworkConnections
```

Counts how many connections each process made every hour.

---

### streamstats

```spl
avg(NetworkConnections)
```

Calculates:

Rolling average.

---

```spl
stdev(NetworkConnections)
```

Calculates:

Standard deviation.

---

### eval

```spl
isOutlier
```

If

```
Current Connections >
Average + 0.5 × Standard Deviation
```

then

```
isOutlier = 1
```

Meaning:

This process is behaving unusually.

---

### search

```spl
search isOutlier=1
```

Returns only suspicious processes.

---

# Why This Works

Normally

```
chrome.exe
20 connections/hour
```

Suddenly

```
chrome.exe
700 connections/hour
```

This becomes an outlier.

Possible reasons:

- Malware
    
- Beaconing
    
- Data exfiltration
    
- C2 communication
    

---

# Detection 1 - Long Command Lines

Attackers often execute extremely long commands.

Example:

- Encoded PowerShell
    
- Base64 payloads
    
- Obfuscated commands
    

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
Image=*cmd.exe
| eval len=len(CommandLine)
| table User len CommandLine
| sort - len
```

---

## len()

Measures command length.

Longer command

↓

More suspicious

---

## Reduce Noise

Exclude normal parents.

```spl
ParentImage!="*msiexec.exe"
ParentImage!="*explorer.exe"
```

Final query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
Image=*cmd.exe
ParentImage!="*msiexec.exe"
ParentImage!="*explorer.exe"
| eval len=len(CommandLine)
| table User len CommandLine
| sort - len
```

---

# Detection 2 - Abnormal cmd.exe Activity

Query:

```spl
index="main"
EventCode=1
(CommandLine="*cmd.exe*")
| bucket _time span=1h
| stats count as cmdCount by _time User CommandLine
| eventstats avg(cmdCount) as avg stdev(cmdCount) as stdev
| eval isOutlier=if(cmdCount>avg+1.5*stdev,1,0)
| search isOutlier=1
```

---

## What Happens?

Calculate

```
Average cmd.exe executions
```

Compare

↓

Current executions

↓

If much higher

↓

Flag as anomaly.

---

# eventstats vs streamstats

|eventstats|streamstats|
|---|---|
|Calculates statistics for the entire result set|Calculates statistics over a moving window|
|Static baseline|Dynamic (rolling) baseline|
|Good for overall averages|Good for time-based anomaly detection|

---

# Detection 3 - Too Many DLLs Loaded

Malware often loads many DLLs quickly.

Sysmon Event:

```
7
```

(Image Loaded)

Query:

```spl
index="main"
EventCode=7
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time Image
| where unique_dlls_loaded>3
| stats count by Image unique_dlls_loaded
```

---

## dc()

```
dc(ImageLoaded)
```

Means

**Distinct Count**

Example

Loaded DLLs

```
kernel32.dll
kernel32.dll
ntdll.dll
user32.dll
```

Distinct count

=

3

---

## Reduce False Positives

Exclude normal locations.

Ignore

```
System32
Program Files
ProgramData
AppData
```

Final query:

```spl
index="main"
EventCode=7
NOT Image="C:\\Windows\\System32*"
NOT Image="C:\\Program Files*"
NOT Image="C:\\Program Files (x86)*"
NOT Image="C:\\ProgramData*"
NOT Image="C:\\Users\\waldo\\AppData*"
| bucket _time span=1h
| stats dc(ImageLoaded) as unique_dlls_loaded by _time Image
| where unique_dlls_loaded>3
| stats count by Image unique_dlls_loaded
| sort - unique_dlls_loaded
```

---

# Detection 4 - Same Process Executed Multiple Times

Sometimes malware repeatedly starts the same process.

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=1
| transaction ComputerName Image
| where mvcount(ProcessGuid)>1
| stats count by Image ParentImage
```

---

## transaction

Groups related events together.

Here:

Same

- Computer
    
- Process
    

---

## mvcount()

```
mvcount(ProcessGuid)
```

Counts

How many process IDs exist.

If

```
>1
```

Means

Process started multiple times.

---

# Investigating Specific Parent-Child Relationships

Example

```
rundll32.exe

↓

svchost.exe
```

Query:

```spl
index="main"
sourcetype="WinEventLog:Sysmon"
EventCode=1
| transaction ComputerName Image
| where mvcount(ProcessGuid)>1
| search Image="C:\\Windows\\System32\\rundll32.exe"
ParentImage="C:\\Windows\\System32\\svchost.exe"
| table CommandLine ParentCommandLine
```

This helps determine whether repeated executions are legitimate or malicious.

---

# Important Splunk Commands

|Command|Purpose|
|---|---|
|`bin` / `bucket`|Group events into time intervals|
|`stats`|Aggregate and summarize data|
|`streamstats`|Running/rolling statistics|
|`eventstats`|Statistics across all matching events|
|`eval`|Create or modify fields|
|`search`|Filter results|
|`where`|Apply conditional filtering|
|`table`|Display selected fields|
|`sort`|Sort results|
|`dc()`|Count distinct values|
|`transaction`|Correlate related events|
|`mvcount()`|Count values in a multivalue field|

---

# Threat Hunting Workflow

```text
Collect Logs
      ↓
Build Normal Baseline
      ↓
Measure Activity
      ↓
Calculate Average
      ↓
Calculate Standard Deviation
      ↓
Identify Outliers
      ↓
Investigate
      ↓
Tune Detection
```

---

# Exam Tips ⭐

- **Analytics-based detection** identifies suspicious activity by finding **statistical anomalies**, not known attack signatures.
    
- **`streamstats`** calculates **rolling statistics** (moving averages and standard deviations).
    
- **`eventstats`** calculates statistics across the **entire result set**.
    
- **`bin`** and **`bucket`** group events into time intervals (e.g., 1 hour).
    
- **`len(CommandLine)`** helps detect unusually long or obfuscated commands.
    
- **`dc(field)`** returns the **distinct count** of values (e.g., unique DLLs loaded).
    
- **`transaction`** groups related events together for correlation.
    
- **`mvcount()`** counts values in a multivalue field and helps identify repeated executions.
    
- Sysmon **Event ID 3** = Network Connections (useful for anomaly detection).
    
- Sysmon **Event ID 7** = DLL/Image Loaded (useful for detecting excessive DLL loading).
    
- Always **reduce false positives** by excluding known benign paths, processes, or parent processes before turning analytics into alerts.



  

# Question 1


### Navigate to http://[Target IP]:8000, open the "Search & Reporting" application, and find through an analytics-driven SPL search against all data the source process images that are creating an unusually high number of threads in other processes. Enter the outlier process name as your answer where the number of injected threads is greater than two standard deviations above the average. Answer format: _.exe

## Solution: 

- By using the knowledge provided in the section, I made up a query:
- ```
  index=* EventCode=8 | stats count as thread_count by SourceImage | eventstats avg(thread_count) as avg_count stdev(thread_count) as stddev_count | eval threshold=avg_count + (2 * stddev_count) | where thread_count > threshold | table SourceImage, thread_count, threshold
  ```
- I got the answer after applying thatttt


