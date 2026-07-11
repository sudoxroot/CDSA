
> **Incident Response reacts to alerts. Threat Hunting actively searches for threats that haven't been detected yet.**

# What is Threat Hunting?

## Traditional Security

Traditional security works like this:

```text
Attacker
     │
     ▼
Attack Happens
     │
     ▼
Antivirus / SIEM Detects
     │
     ▼
Security Team Responds
```

This is **reactive**.

The defender waits until something triggers an alert.

---

## Threat Hunting

Threat hunting works differently.

```text
Attacker
     │
     ▼
May already exist
     │
     ▼
Hunter actively searches
     │
     ▼
Find hidden attacker
```

Instead of waiting for an alert,

the hunter asks:

> "Is someone already inside our network?"

---

# What is Dwell Time?

One of the most important CPTS terms.

**Dwell Time**

= The amount of time an attacker stays inside a network before being detected.

Example:

```text
Day 1
Attacker enters

↓

Day 25

Security discovers attacker
```

Dwell Time = **24 days**

---

## Why is long dwell time dangerous?

### Detected immediately

```text
Enter

↓

Caught
```

Little damage.

---

### Detected after three weeks

```text
Enter

↓

Steal documents

↓

Install cameras

↓

Copy passwords

↓

Destroy evidence

↓

Finally detected
```

Huge damage.

Threat hunting aims to **reduce dwell time**.

---

# Definition of Threat Hunting

Threat hunting is:

- Active
    
- Human-led
    
- Hypothesis-driven
    
- Proactive
    

Notice:

It is **not automatic**.

Humans perform threat hunting using their experience.

---

# Hypothesis-Driven Hunting

A hunter starts with a hypothesis.

Example:

> "Attackers often use PowerShell with Base64 encoded commands."

Then they search:

```text
PowerShell

AND

-EncodedCommand
```

If they find matches,

they investigate further.

---

Another example:

> "Ransomware usually uses PsExec."

Search:

```text
PsExec.exe

↓

Remote execution

↓

Lateral movement
```

---

# Main Goal

Reduce dwell time.

The earlier attackers are discovered,

the less damage they can cause.

---

# Threat Hunting Process

The notes describe a simple workflow.

## Step 1

Identify important assets.

Example:

```text
Domain Controller

File Server

SQL Database

HR Server
```

These are high-value targets.

---

## Step 2

Study attacker behavior.

Use Threat Intelligence.

Learn

- Tactics
    
- Techniques
    
- Procedures (TTPs)
    

---

## Step 3

Create hypotheses.

Example

> Attackers often abuse scheduled tasks.

---

## Step 4

Search for evidence.

Look for:

- Event Logs
    
- Sysmon
    
- ETW
    
- Network Traffic
    
- Registry
    
- Scheduled Tasks
    

---

## Step 5

Validate findings.

Ask

> Is this normal?

or

> Is this malicious?

---

# What is Threat Intelligence?

Threat Intelligence is information about attackers.

Examples include:

- Known malware
    
- Known attacker IPs
    
- Command-and-Control domains
    
- MITRE ATT&CK techniques
    
- Indicators of Compromise (IoCs)
    

Think of it as:

```text
Threat Reports

↓

Hunter

↓

Better hypotheses
```

---

# Important Characteristics of Threat Hunting

## 1. Offensive

Instead of waiting,

go searching.

---

## 2. Hypothesis-based

Every hunt starts with an idea.

Example:

> Attackers often use `rundll32.exe`.

Search for:

```text
rundll32.exe

↓

Unusual DLL

↓

Suspicious Command Line
```

---

## 3. Know the attacker

Understand

- TTPs
    
- MITRE ATT&CK
    
- Malware
    
- Kill Chain
    

---

## 4. Know your own environment

This is extremely important.

You must know what is **normal**.

Otherwise,

everything looks suspicious.

Example

```text
SQL Server

normally uses port 1433
```

Not suspicious.

---

Example

```text
SQL Server

starts cmd.exe

downloads PowerShell script
```

Very suspicious.

---

# Threat Hunting vs Incident Response

People often confuse these.

## Incident Response

```text
Alert

↓

Investigate

↓

Contain

↓

Recover
```

Reactive.

---

## Threat Hunting

```text
No Alert

↓

Search anyway

↓

Find attacker
```

Proactive.

---

# Relationship with Incident Handling

Threat hunting helps during every incident phase.

---

## Preparation

Hunters help create

- procedures
    
- rules
    
- playbooks
    

---

## Detection & Analysis

Hunters investigate alerts.

They often discover

- additional malware
    
- persistence
    
- lateral movement
    

---

## Containment

Some organizations allow hunters to help isolate infected systems.

Others leave that to Incident Response.

---

## Recovery

Hunters verify

> Is every attacker artifact gone?

---

## Lessons Learned

Hunters recommend

- better detections
    
- new SIEM rules
    
- better logging
    
- stronger monitoring
    

---

# Threat Hunting Team

Threat hunting is a team effort.

## Threat Hunter

Main investigator.

Searches for attackers.

---

## Threat Intelligence Analyst

Studies:

- APT groups
    
- IoCs
    
- MITRE ATT&CK
    
- Malware reports
    

Provides intelligence.

---

## Incident Responder

Handles

- containment
    
- eradication
    
- recovery
    

---

## Forensics Expert

Investigates

- malware
    
- disks
    
- memory
    
- timelines
    

---

## Data Scientist

Finds patterns in huge datasets.

Example

Millions of Sysmon logs.

---

## Security Engineer

Builds

- SIEM
    
- EDR
    
- detection rules
    
- logging
    

---

## Network Security Analyst

Looks at

- DNS
    
- HTTP
    
- SMB
    
- unusual traffic
    

---

## SOC Manager

Coordinates everyone.

---

# When Should We Hunt?

The notes mention five situations.

---

## 1. New Vulnerability

Example

A new Windows vulnerability is published.

Question:

> Has anyone exploited it inside our company?

---

## 2. New Threat Intelligence

Example

Microsoft publishes

```text
New C2 IPs

↓

New malware hash

↓

New attacker domains
```

Search immediately.

---

## 3. Multiple Anomalies

One strange event

↓

Maybe nothing.

Ten strange events

↓

Start hunting.

---

## 4. During Incident Response

Suppose ransomware is found on one computer.

Don't only investigate that machine.

Hunt across the entire network.

Maybe

```text
PC 1

PC 2

Server

Domain Controller
```

are also infected.

---

## 5. Regular Hunting

Even if nothing appears wrong,

hunt regularly.

Think of it like a health check-up.

---

# Risk Assessment & Threat Hunting

Risk Assessment tells us

> "Where should we look first?"

---

Suppose the company has

```text
Web Server

↓

Public

↓

High Risk
```

and

```text
Printer

↓

Low Risk
```

Where should hunters spend more time?

Obviously,

the web server.

---

# Risk Assessment Helps Hunters

## Identify Crown Jewels

Examples

- Domain Controller
    
- Database
    
- HR Server
    
- Payment Server
    

Protect these first.

---

## Understand Threats

Learn which attackers target your organization.

---

## Find Weaknesses

Example

Old VPN server

↓

Known vulnerability

↓

Search for exploitation.

---

## Use Threat Intelligence Better

If intelligence says

> APT29 targets Exchange Servers

You should hunt your Exchange servers.

---

## Improve Incident Response

Knowing the biggest risks helps create better response plans.

---

## Improve Security Controls

After hunting,

organizations may:

- Enable Sysmon
    
- Collect ETW logs
    
- Add SIEM rules
    
- Improve EDR
    
- Patch vulnerabilities
    

---

# Threat Hunting Workflow

```text
Risk Assessment
        │
        ▼
Identify Critical Assets
        │
        ▼
Threat Intelligence
        │
        ▼
Develop Hunting Hypothesis
        │
        ▼
Collect Logs
(Sysmon, ETW, Windows Logs, Network)
        │
        ▼
Analyze
        │
        ▼
Find IoCs / TTPs
        │
        ▼
Validate
        │
        ▼
Incident Response (if needed)
        │
        ▼
Improve Detection Rules
```

---

# Quick Revision 📝

- **Threat Hunting** = Proactively searching for hidden attackers before alerts are triggered.
    
- **Dwell Time** = Time between an attacker's initial compromise and their detection. The goal is to reduce it.
    
- **Threat hunting is**:
    
    - Human-led
        
    - Hypothesis-driven
        
    - Intelligence-driven
        
    - Proactive
        
- **Typical workflow**:
    
    1. Identify critical assets.
        
    2. Study attacker TTPs using Threat Intelligence.
        
    3. Create a hunting hypothesis.
        
    4. Search logs and telemetry (Sysmon, ETW, Windows Events, network data).
        
    5. Validate findings and respond if necessary.
        
- **Threat Hunting vs Incident Response**:
    
    - **Threat Hunting**: Searches for unknown threats without waiting for alerts.
        
    - **Incident Response**: Investigates and contains known security incidents.
        
- **Common triggers for threat hunting**:
    
    - New vulnerabilities
        
    - New threat intelligence or IoCs
        
    - Multiple suspicious anomalies
        
    - During an active incident
        
    - Regular proactive security reviews
        
- **Risk Assessment** helps prioritize hunting by identifying high-value assets ("crown jewels"), understanding likely threats, highlighting vulnerabilities, and improving security controls.

-   
    
    ## Question 1
  
    ### Threat hunting is used ... Choose one of the following as your answer: "proactively", "reactively", "proactively and reactively".
    
    ## Solution:
    - **Answer:** proactively
    
- ## Question 2
      
    ### Threat hunting and incident handling are two processes that always function independently. Answer format: True, False.
    
    ## Solution:
    - **Answer:** False
    
- ## Question 3
    
    ### Threat hunting and incident response can be conducted simultaneously. Answer format: True, False.
    ## Solution:
    - **Answer:** True
