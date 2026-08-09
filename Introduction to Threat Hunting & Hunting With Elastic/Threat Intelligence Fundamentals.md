

# Cyber Threat Intelligence (CTI)

## Definition

**Cyber Threat Intelligence (CTI)** is the process of **collecting, analyzing, and sharing information about cyber threats** so defenders can stop attacks **before or during** an attack.

### Goal

Move security from:

> **Reactive → Proactive**

Instead of waiting for an attack, CTI tries to predict and prepare for it.

Think:

```
Logs + Threat Reports + Analysis
            ↓
     Threat Intelligence
            ↓
 Better Detection & Defense
```

---

# Four Principles of Good CTI

Good CTI must satisfy **four requirements**.

## 1. Relevance

**Question: Does this information matter to us?**

Not every vulnerability is important.

Example

- Your company uses Windows.
    
- A vulnerability is found in macOS.
    

This intelligence is **not very relevant**.

### Remember

> Relevant = Useful for YOUR organization.

---

## 2. Timeliness

**Question: Is the information still fresh?**

Threat intelligence loses value over time.

Example

- Yesterday's phishing domain may already be offline.
    
- Malware C2 server may already be changed.
    

Fresh intelligence is much more valuable.

### Remember

> Older intelligence = Less useful.

---

## 3. Actionability

**Question: Can defenders do something with it?**

Good intelligence tells defenders exactly what to do.

Example

Instead of saying

```
A ransomware group exists.
```

Better intelligence says

```
Block IP X
Monitor Domain Y
Detect PowerShell command Z
```

Now defenders can act.

---

### "Self-Licking Ice Cream Cone"

Funny phrase but important.

It means:

> Analysts keep analyzing information that never produces useful actions.

Lots of work.

No improvement.

Avoid this.

---

## 4. Accuracy

Incorrect intelligence wastes time.

Example

Suppose intelligence says

```
192.168.5.8
is malicious
```

You block it.

Later you discover it belongs to your own server.

Bad intelligence caused damage.

Always verify information.

---

## Easy Memory

```
RATA

R → Relevant

A → Actionable

T → Timely

A → Accurate
```

---

# Benefits of CTI

Good CTI helps organizations:

- Predict attacks
    
- Understand attackers
    
- Improve detections
    
- Build better defenses
    
- Help executives make decisions
    

---

# Threat Intelligence vs Threat Hunting

Students confuse these constantly.

|Threat Intelligence|Threat Hunting|
|---|---|
|Predictive|Proactive + Reactive|
|Studies attackers|Searches your network|
|Answers "What may happen?"|Answers "Is it happening?"|
|Builds attacker profiles|Finds attackers|

---

## Threat Intelligence asks

- Who?
    
- Where?
    
- When?
    
- Why?
    
- What techniques?
    

---

## Threat Hunting asks

```
Is this attacker already inside?

Did we miss something?

Can we find evidence?
```

---

## Relationship

Threat Intelligence

↓

Produces IOCs

↓

Threat Hunters search for them

↓

Hunters discover new evidence

↓

CTI team improves intelligence

It's a continuous feedback loop.

---

# CTI Criteria

Good CTI provides

- Threat visibility
    
- Awareness
    
- Better understanding
    
- Reduced organizational risk
    

Leadership uses CTI to decide:

- Should we patch immediately?
    
- Should we buy new security tools?
    
- Should we change business processes?
    

---

# Three Types of Threat Intelligence

Very important for CPTS.

```
Strategic

↓

Operational

↓

Tactical
```

As you move downward:

- Less management
    
- More technical detail
    

---

# 1. Strategic Intelligence

Audience

- CEO
    
- CISO
    
- Executives
    
- Leadership
    

Focus

Business risk.

Questions answered

```
Who?

Why?
```

Contains

- Nation-state activity
    
- Industry threats
    
- Long-term trends
    
- Business impact
    

---

Example

```
APT28

Targets governments.

Motivation:
Political espionage.
```

Executives decide risk.

---

# 2. Operational Intelligence

Audience

- SOC Managers
    
- Security Managers
    

Focus

Campaign analysis.

Questions answered

```
How?

Where?
```

Contains

- Campaign details
    
- TTPs
    
- Attack methods
    
- Infrastructure
    

---

Example

```
REvil

Initial access

↓

Phishing

↓

Credential dumping

↓

Ransomware deployment
```

---

# 3. Tactical Intelligence

Audience

SOC Analysts

Threat Hunters

Incident Responders

Focus

Immediate action.

Contains

Technical indicators.

Examples

- IP addresses
    
- Domains
    
- URLs
    
- File hashes
    
- Registry keys
    
- Mutex names
    
- YARA rules
    
- C2 servers
    

Question answered

```
What do I block today?
```

---

# Easy Comparison

|Strategic|Operational|Tactical|
|---|---|---|
|Executives|Managers|Analysts|
|Business|Campaigns|Technical details|
|Who & Why|How & Where|What to detect/block|

---

# Venn Diagram Meaning

All three intelligence types overlap.

Example

Strategic says

```
APT29 targets healthcare.
```

↓

Operational says

```
Uses spear phishing.
```

↓

Tactical says

```
Block these IPs.

Detect this hash.

Monitor this registry key.
```

Together they give a complete picture.

---

# Reading a Tactical Threat Intelligence Report

Whenever you receive a CTI report, follow this workflow.

---

## Step 1 : Understand the Story

Read the report first.

Ask:

- Who is attacking?
    
- Who is the target?
    
- Why?
    

Example

```
Emotet targeting finance companies.
```

---

## Step 2 : Extract the IOCs

Separate them.

### Network IOCs

- IPs
    
- Domains
    
- URLs
    
- DNS
    

---

### Host IOCs

- Hashes
    
- Registry keys
    
- DLLs
    
- File paths
    
- Processes
    

---

### Email IOCs

- Sender
    
- Subject
    
- Attachments
    

---

Example

```
Network

↓

185.10.x.x

evil.com

↓

Host

↓

SHA256

Registry Key

↓

Email

↓

Invoice Attached
```

---

## Step 3 : Understand the Attack Lifecycle

Don't just collect IOCs.

Understand **how the attack works**.

Example mapped to the MITRE ATT&CK framework:

```
Phishing

↓

Execution

↓

Persistence

↓

Defense Evasion

↓

Command & Control

↓

Data Theft
```

Knowing the attack chain helps you detect variants, not just exact IOCs.

---

## Step 4 : Validate IOCs

Never trust every IOC blindly.

Check:

- Is it still active?
    
- Is it accurate?
    
- Is it a false positive?
    
- Does another intelligence source confirm it?
    

Remember

An IP may belong to cloud hosting where legitimate services also run.

Context matters.

---

## Step 5 : Deploy the IOCs

Now integrate them into security tools.

Examples

Firewall

```
Block malicious IP
```

EDR

```
Detect file hash
```

IDS/IPS

```
Detect malicious traffic
```

Email Gateway

```
Block phishing sender
```

Always consider business impact before blocking.

---

## Step 6 : Hunt for Evidence

Now become a threat hunter.

Search:

- Firewall logs
    
- DNS logs
    
- Proxy logs
    
- Windows Event Logs
    
- PowerShell logs
    
- Endpoint telemetry
    

Don't only search for exact IOCs.

Also search for attacker behavior (TTPs).

Example

Instead of searching only for one malware hash, look for suspicious PowerShell execution or unusual persistence mechanisms that match the campaign.

---

## Step 7 : Monitor and Improve

After deployment:

- Watch for alerts.
    
- Trigger incident response if an IOC is detected.
    
- Improve detection rules.
    
- Share newly discovered IOCs with trusted intelligence-sharing communities.
    

This creates a continuous cycle of learning and stronger defenses.

---

# Complete CTI Workflow

```
Threat Reports
        │
        ▼
Understand the Campaign
        │
        ▼
Extract IOCs
        │
        ▼
Study TTPs
        │
        ▼
Validate Indicators
        │
        ▼
Deploy to Security Tools
        │
        ▼
Threat Hunt
        │
        ▼
Monitor Alerts
        │
        ▼
Improve Intelligence
```

# Exam Tips

- Memorize the **RATA** qualities of good CTI: **Relevant, Actionable, Timely, Accurate**.
    
- Know the difference between **Threat Intelligence** (predicts threats) and **Threat Hunting** (looks for evidence in your environment).
    
- Be able to distinguish the three intelligence levels:
    
    - **Strategic** → Executives, business risk, **Who/Why**
        
    - **Operational** → Managers, campaigns, **How/Where**
        
    - **Tactical** → Analysts, technical IOCs, **What to detect or block**
        
- When reading a CTI report, think in order: **Understand → Extract IOCs → Analyze TTPs → Validate → Deploy → Hunt → Monitor**.
    
- Don't rely only on IOCs. Skilled threat hunters also look for **behaviors (TTPs)** because attackers can easily change hashes, IPs, and domains.


-   
    
    ## Question 1

    
    ### It’s useful for the CTI team to provide a single IP with no context to the SOC team. Answer format: True, False.
    - **Answer:** False
    
- ## Question 2

    
    ### When an incident occurs on the network and the CTI team is made aware, what should they do? Choose one of the following as your answer: "Do Nothing", "Reach out to the Incident Handler/Incident Responder", "Provide IOCs on all research being conducted, regardless if the IOC is verified".
    - **Answer:** Reach out to the incident Handle/Incident Responder
- ## Question 3

    ### When an incident occurs on the network and the CTI team is made aware, what should they do? Choose one of the following as your answer: "Provide IOCs on all research being conducted, regardless if the IOC is verified", "Do Nothing", "Provide further IOCs and TTPs associated with the incident".
    - **Answer:** Provide further IOCs and TTPs associated with the incident
- ## Question 4
    

    ### Cyber Threat Intelligence, if curated and analyzed properly, can ... ? Choose one of the following as your answer: "be used for security awareness", "be used for fine-tuning network segmentation", "provide insight into adversary operations".
    - **Answer:** provide insight into adversary operations














