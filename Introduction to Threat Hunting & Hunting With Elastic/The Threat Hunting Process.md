
# Threat Hunting Process (Simple Explanation)

Think of threat hunting like a detective investigating a serial burglar.

The detective doesn't randomly search houses.

Instead, they follow a structured investigation.

```text
Prepare
    ↓
Create a hypothesis
    ↓
Plan the investigation
    ↓
Collect evidence
    ↓
Analyze evidence
    ↓
Respond
    ↓
Improve
    ↓
Repeat
```

---

# Step 1: Setting the Stage (Preparation)

## Goal

Prepare everything before hunting.

Think of it as:

> "Before looking for a criminal, make sure you have cameras, fingerprints, maps, and police officers ready."

---

## What happens here?

The security team asks:

- What assets are important?
    
- Who may attack us?
    
- What tools do we have?
    
- Are logs enabled?
    
- Are SIEM and EDR collecting data?
    

---

### Example

Company assets:

```
Domain Controller

Email Server

HR Database

Finance Server
```

These become priority targets.

---

### Threat Intelligence

The team researches:

- Latest malware
    
- New APT campaigns
    
- New vulnerabilities
    
- MITRE ATT&CK techniques
    

Example:

```
Threat Intel says:

Emotet now spreads using Excel macros.
```

Now the hunters know what to look for.

---

### Logging

Without logs...

```text
No Logs
↓

No Evidence
↓

No Hunting
```

Good hunters ensure:

- Windows Event Logs
    
- Sysmon
    
- ETW
    
- Network Logs
    
- DNS Logs
    
- Proxy Logs
    

are all being collected.

---

## Remember

Preparation =

> "Make sure visibility exists before hunting."

---

# Step 2: Formulating Hypotheses

This is the most important step.

Threat hunting is **hypothesis-driven**.

---

Instead of randomly searching,

you ask:

> "If attackers are here, how would they behave?"

---

Example hypothesis

```
Attackers are using phishing emails
to deliver Emotet.
```

Another:

```
Attackers use PowerShell
with EncodedCommand.
```

Another:

```
Attackers establish C2
through DNS tunneling.
```

Every hypothesis must be:

- Specific
    
- Testable
    

---

Bad hypothesis

```
Hackers are attacking us.
```

Impossible to test.

---

Good hypothesis

```
Emotet is using malicious Word
documents containing macros.
```

Very specific.

---

# Step 3: Designing the Hunt

Now decide

**Where do we search?**

Think like a detective.

If someone stole a car,

you don't search the ocean.

You search:

- Roads
    
- Cameras
    
- Garages
    

Same idea.

---

## Choose data sources

Examples

```
Email Logs

DNS Logs

Firewall Logs

Sysmon

Windows Event Logs

Proxy Logs

EDR
```

---

Choose tools

Examples

```
Splunk

Elastic

Sentinel

Sysmon

Wireshark

Chainsaw

Velociraptor
```

---

Choose IoCs

Examples

```
IP Address

Hash

Domain

Filename

Registry Key

Mutex

User-Agent
```

---

Example

Hypothesis:

```
Emotet uses C2 server
203.0.113.55
```

Search

```
Firewall logs

↓

Any communication with

203.0.113.55
```

---

# Step 4: Data Gathering & Examination

Now the real hunting begins.

Collect evidence.

Examples:

```
Sysmon Logs

DNS Logs

Email Logs

Windows Logs

Network Packets

Memory

EDR
```

---

Then analyze.

Example

Normal:

```
Chrome.exe

↓

Google.com
```

Nothing suspicious.

---

Suspicious

```
Word.exe

↓

PowerShell

↓

cmd.exe

↓

Unknown IP
```

Very suspicious.

---

This phase is iterative.

Sometimes

```
Hypothesis

↓

Evidence

↓

New hypothesis

↓

More evidence
```

You continuously refine your investigation.

---

# Step 5: Evaluate Findings

Now ask

Did we prove the hypothesis?

---

Example

Hypothesis

```
Emotet is using phishing emails.
```

Evidence

```
✓ Malicious Word file

✓ Macro execution

✓ PowerShell

✓ C2 communication
```

Hypothesis confirmed.

---

Another example

```
No macro

No PowerShell

No C2

No malware
```

Hypothesis rejected.

---

Evaluate

- How many systems?
    
- Which users?
    
- What was stolen?
    
- How serious?
    

---

# Step 6: Mitigate Threats

Now remove the threat.

Possible actions

```
Disconnect infected PC

↓

Kill malicious process

↓

Delete malware

↓

Reset passwords

↓

Patch vulnerability

↓

Block IP

↓

Block domain
```

Goal:

Stop the attacker immediately.

---

Example

Compromised workstation

↓

Disconnect network cable

↓

Remove Emotet

↓

Reset user's password

↓

Patch Office

---

# Step 7: After the Hunt

Never stop after removing malware.

Document everything.

---

Document

- Timeline
    
- IoCs
    
- TTPs
    
- Malware
    
- Detection methods
    
- Root Cause
    

---

Improve detections

Example

Create new Sysmon rule

```
Alert when

Word.exe

↓

starts PowerShell
```

Future attacks are detected faster.

---

Update playbooks

```
Emotet Playbook

↓

Step 1

Isolate PC

↓

Step 2

Collect memory

↓

Step 3

Remove malware
```

---

# Step 8: Continuous Learning

Threat hunting never ends.

Attackers evolve.

Hunters must evolve too.

Every hunt improves

- Detection rules
    
- SIEM queries
    
- Playbooks
    
- Skills
    
- Logging
    

Think of it like:

```text
Hunt

↓

Learn

↓

Improve

↓

Better Hunt

↓

Learn Again
```

---

# Emotet Example (Complete Flow)

Let's apply all 8 steps.

---

## Step 1

Preparation

Research says:

```
Emotet spreads

↓

Phishing emails

↓

Word macros
```

Enable

- Email Logs
    
- Sysmon
    
- DNS Logs
    
- EDR
    

---

## Step 2

Hypothesis

```
Someone received

an Emotet phishing email.
```

---

## Step 3

Design

Search

```
Email Logs

↓

.doc attachments

↓

Macros

↓

PowerShell

↓

Known Emotet domains
```

---

## Step 4

Collect

You discover

```
invoice.doc

↓

WINWORD.exe

↓

powershell.exe

↓

Unknown IP
```

---

## Step 5

Evaluate

Evidence confirms

```
Macro executed

↓

PowerShell launched

↓

Connected to Emotet C2
```

Hypothesis confirmed.

---

## Step 6

Mitigate

```
Disconnect PC

↓

Delete malware

↓

Reset credentials

↓

Patch Office

↓

Block C2 IP
```

---

## Step 7

Document

Record

```
Hash

IP

Domains

Registry Keys

Scheduled Tasks

Persistence
```

---

## Step 8

Improve

Add SIEM rules

Example

```
Alert if

WINWORD.exe

↓

powershell.exe
```

Now future Emotet attacks are detected immediately.

---

# Complete Threat Hunting Flow

```text
Threat Intelligence
        │
        ▼
Prepare Environment
(Log Collection, SIEM, EDR)
        │
        ▼
Create Hypothesis
        │
        ▼
Design Hunt
(Data Sources + IoCs + Queries)
        │
        ▼
Collect Logs
(Sysmon, ETW, Windows Logs, DNS, Network)
        │
        ▼
Analyze Evidence
        │
        ▼
Hypothesis Confirmed?
       / \
     No   Yes
     │      │
Refine      ▼
Hypothesis Mitigate Threat
             │
             ▼
Document Findings
             │
             ▼
Improve Detection Rules
             │
             ▼
Start Next Hunt
```

# Exam Notes 📝

- **Threat hunting is proactive**, not reactive.
    
- Every hunt begins with a **testable hypothesis**.
    
- **Preparation** focuses on visibility: enable logging, configure SIEM/EDR/IDS, identify critical assets, and study threat intelligence.
    
- **Designing the hunt** answers:
    
    - What data will I search?
        
    - Which tools will I use?
        
    - Which IoCs/TTPs am I looking for?
        
- **Data gathering** commonly uses Sysmon, Windows Event Logs, ETW, EDR telemetry, DNS logs, email logs, firewall logs, and network captures.
    
- **Evaluation** determines whether the evidence confirms or disproves the hypothesis.
    
- **Mitigation** includes isolating hosts, removing malware, patching vulnerabilities, blocking malicious infrastructure, and resetting compromised credentials.
    
- Every hunt should produce **new knowledge** that improves detection rules, playbooks, and future hunting capabilities.
    
- Remember the flow:  
    **Prepare → Hypothesize → Design → Collect → Analyze → Evaluate → Mitigate → Document → Improve → Repeat**
    

  

## Question 1


### It is OK to formulate hypotheses that are not testable. Answer format: True, False.

**Answer:** False