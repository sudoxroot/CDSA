

# Part 1 - What is Splunk?

Imagine you own a company with:

- 100 Windows PCs
    
- Linux servers
    
- Firewalls
    
- Routers
    
- Antivirus
    
- Web servers
    

Every device creates **logs**.

Without Splunk:

```
PC1 → Logs
PC2 → Logs
Firewall → Logs
Server → Logs
```

Everything is scattered.

Splunk collects everything into **one place**.

```
All Devices
      ↓
   Splunk
      ↓
 Search
 Analyze
 Detect attacks
 Build Dashboards
```

Think of Splunk as:

> **Google for logs.**

---

# Why do companies use Splunk?

Splunk helps with

- Security Monitoring
    
- Threat Hunting
    
- Incident Response
    
- Compliance
    
- System Monitoring
    
- Business Analytics
    

For CDSA, remember:

> Splunk is mainly a SIEM.

---

# Part 2 - Splunk Architecture

There are three major components.

```
Device
   ↓
Forwarder
   ↓
Indexer
   ↓
Search Head
```

---

## 1. Forwarder

A Forwarder collects logs.

Think:

```
Forwarder = Delivery Boy
```

It goes to computers, picks up logs, and delivers them.

Example

```
Windows PC
      ↓
Forwarder
      ↓
Splunk
```

---

### Types of Forwarders

## Universal Forwarder (UF)

Small and lightweight.

It only sends logs.

No processing.

Example

```
PC
 ↓
Universal Forwarder
 ↓
Indexer
```

Remember:

> Collects and forwards only.

---

## Heavy Forwarder (HF)

Heavy Forwarder is smarter.

It can

- filter logs
    
- modify logs
    
- route logs
    
- parse logs
    

Example

```
Firewall
 ↓
Heavy Forwarder
 ↓
Filters Logs
 ↓
Indexer
```

Think

```
UF = Courier

HF = Courier + Manager
```

---

## HTTP Event Collector (HEC)

Some applications don't use forwarders.

Instead they directly send logs.

```
Application

↓

HTTP API

↓

Indexer
```

No Forwarder required.

---

# Part 3 - Indexer

Indexer is the brain.

It receives logs.

Then it

- compresses logs
    
- indexes logs
    
- stores logs
    
- answers searches
    

Example

```
Logs

↓

Indexer

↓

Database
```

Think

```
Google indexes websites.

Splunk indexes logs.
```

---

# Part 4 - Search Head

Users never search the Indexer directly.

They use Search Head.

```
You

↓

Search Head

↓

Indexer

↓

Results
```

Search Head is basically

> Splunk Web Interface

---

# Other Components

Deployment Server

Used to configure many Forwarders.

Instead of configuring 500 forwarders one by one.

---

Cluster Master

Controls multiple Indexers.

Makes sure copies of data exist.

---

License Master

Manages Splunk licenses.

---

# Architecture Summary

```
Logs

↓

Forwarder

↓

Indexer

↓

Search Head

↓

Analyst
```

---

# Part 5 - SPL

SPL

=

Search Processing Language

Think

```
SQL

→ Database

SPL

→ Splunk
```

Everything in Splunk is searched using SPL.

---

# Part 6 - Basic Search

Example

```spl
index="main"
```

Meaning

Search everything inside the **main** index.

---

Example

```spl
index="main" error
```

Meaning

Find every event containing

```
error
```

---

Example

```spl
index="main" "UNKNOWN"
```

Find logs containing

```
UNKNOWN
```

---

# Wildcards

```
*
```

means

"anything"

Example

```spl
*UNKNOWN*
```

Matches

```
UNKNOWN

UNKNOWN_USER

USER_UNKNOWN

UNKNOWN123
```

---

# Boolean Operators

Exactly like programming.

AND

```spl
error AND powershell
```

Must contain both.

---

OR

```spl
error OR warning
```

Either one.

---

NOT

```spl
NOT powershell
```

Exclude PowerShell.

---

# Part 7 - Fields

Every log has fields.

Example

```
Time

Host

User

Process

EventCode

CommandLine
```

Example

```
User = Administrator

Host = DC01

EventCode = 4624
```

---

Comparison Operators

```
=
!=
<
>
<=
>=
```

Example

```spl
EventCode!=1
```

Means

Show everything except Event ID 1.

---

# Part 8 - fields command

Suppose logs contain

```
Time

Host

User

Image

CommandLine
```

You don't want User.

```
| fields - User
```

Result

```
Time

Host

Image

CommandLine
```

Think

```
Hide columns.
```

---

# Part 9 - table command

```
| table host Image User
```

Instead of showing every field,

show only

```
Host

Image

User
```

Think

```
Excel table.
```

---

# Part 10 - rename

```spl
rename Image as Process
```

Now

```
Image
```

becomes

```
Process
```

Only the name changes.

The data stays the same.

---

# Part 11 - dedup

Suppose

```
chrome.exe

chrome.exe

chrome.exe

cmd.exe
```

Using

```spl
dedup Image
```

Result

```
chrome.exe

cmd.exe
```

Duplicates removed.

---

# Part 12 - sort

Example

```spl
sort - _time
```

"-"

means descending.

Newest first.

```
Newest

↓

Oldest
```

---

# Part 13 - stats

One of the most important SPL commands.

It performs calculations.

Example

```spl
stats count by Image
```

Result

```
chrome.exe      120

powershell.exe   8

cmd.exe         22
```

Think

```
GROUP BY
```

from SQL.

---

# Part 14 - chart

Very similar to stats.

Difference

```
stats

↓

Numbers

chart

↓

Graphs
```

Perfect for dashboards.

---

# Part 15 - eval

Creates a new field.

Example

```spl
eval username=lower(User)
```

If

```
ADMIN
```

Result

```
admin
```

Original field stays unchanged.

---

# Part 16 - rex

rex

=

Regular Expressions

Used to extract text.

Example

```
C:\Windows\cmd.exe
```

Extract

```
cmd.exe
```

Think

```
Regex extraction.
```

---

# Part 17 - lookup

Suppose you have

|filename|is_malware|
|---|---|
|cmd.exe|false|
|notepad.exe|false|
|sharphound.exe|true|

Now Splunk compares every process against this CSV.

If it finds

```
sharphound.exe
```

Output

```
is_malware = true
```

Think

```
Excel VLOOKUP.
```

---

# Part 18 - inputlookup

Instead of searching logs,

show the CSV itself.

```spl
| inputlookup malware_lookup.csv
```

Displays

```
filename

is_malware
```

---

# Part 19 - Time Filtering

```spl
earliest=-7d
```

Means

```
Last 7 days
```

Examples

```
-24h

Last day

-30m

Last 30 minutes

-1h

Last hour
```

---

# Part 20 - transaction

One of the most powerful commands.

Groups related events together.

Example

```
Process Started

↓

Network Connection

↓

Registry Change
```

Instead of three separate events

Splunk shows

```
One Transaction
```

Useful for tracking attacker activity.

---

# Part 21 - Subsearch

Search inside another search.

Example

```
Outer Search

↓

Inner Search

↓

Results used by Outer Search
```

Think

```
SQL Subquery
```

---

# Part 22 - Discovering Available Data

Before hunting threats, you need to know:

- What indexes exist?
    
- What log sources are available?
    
- What fields exist?
    

Useful commands:

Show indexes:

```spl
| eventcount summarize=false index=* | table index
```

Show all sourcetypes:

```spl
| metadata type=sourcetypes
```

Show data sources:

```spl
| metadata type=sources
```

Show raw logs:

```spl
sourcetype="WinEventLog:Security" | table _raw
```

Show all fields (use cautiously because output can be very wide):

```spl
sourcetype="WinEventLog:Security" | table *
```

Show field summary:

```spl
sourcetype="WinEventLog:Security" | fieldsummary
```

Find rare events:

```spl
index=* sourcetype=* | rare limit=10 index, sourcetype
```

---

# Part 23 - Using the Splunk UI

Besides SPL, you can discover data through the interface:

- **Data Inputs**: See where logs come from (files, scripts, forwarders, HEC, etc.).
    
- **Search & Reporting**: Browse events. Use **Fast** mode for quick searches and **Verbose** mode to inspect every extracted field.
    
- **Selected Fields**: Common fields always shown (e.g., `host`, `source`, `sourcetype`).
    
- **Interesting Fields**: Fields appearing in many events (about 20% or more).
    
- **All Fields**: Displays every extracted field for an event.
    

---

# Part 24 - Data Models & Pivots

### Data Models

Data Models organize raw logs into logical categories, making searches and reports easier.

Example:

```
Web Traffic
├── URL
├── Status Code
└── User
```

They allow analysts to build reports without writing complex SPL.

### Pivots

Pivots provide a drag-and-drop interface to build reports, charts, and dashboards from Data Models.

Think of it as:

```
Excel Pivot Table
        +
Splunk Data
```

No SPL knowledge is required for basic Pivot reports.

---

# 📚 Revision Notes

|Component|Remember|
|---|---|
|**Splunk**|Google for logs / SIEM platform|
|**Universal Forwarder (UF)**|Collects and forwards logs only|
|**Heavy Forwarder (HF)**|Collects, parses, filters, routes logs|
|**HTTP Event Collector (HEC)**|Applications send logs directly via HTTP|
|**Indexer**|Stores, compresses, indexes, and searches logs|
|**Search Head**|User interface for searching and dashboards|
|**SPL**|Splunk Search Processing Language|
|**fields**|Hide or keep specific fields|
|**table**|Display selected columns|
|**rename**|Rename a field|
|**dedup**|Remove duplicates|
|**sort**|Sort results (e.g., newest first)|
|**stats**|Aggregate data (counts, sums, averages)|
|**chart**|Create chart-friendly aggregated output|
|**eval**|Create or modify fields|
|**rex**|Extract values using regular expressions|
|**lookup**|Match data against an external CSV (like VLOOKUP)|
|**inputlookup**|Display the contents of a lookup table|
|**earliest/latest**|Restrict searches to a time range|
|**transaction**|Group related events into one activity|
|**Subsearch**|Run a search inside another search|
|**fieldsummary**|Summarize all available fields|
|**rare**|Find uncommon values or events|
|**Data Models**|Organize data into logical structures|
|**Pivots**|Build reports and dashboards without SPL|

These are the commands and concepts that appear repeatedly in HTB Academy's Splunk modules and are the ones you should be comfortable recognizing and using during CDSA labs and the certification exam.





