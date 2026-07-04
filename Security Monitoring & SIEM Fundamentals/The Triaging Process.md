This is one of the **most important SOC concepts** in CPTS. Almost every alert you investigate follows this workflow. Instead of memorizing the long list, understand **why each step exists**. Here's a simplified explanation with a practical SOC perspective.

---

# What Is Alert Triaging?

**Alert triaging** is the process of deciding:

- **Is this alert real?**
    
- **How dangerous is it?**
    
- **What should we do next?**
    
- **Who should handle it?**
    

Think of a hospital emergency room (ER).

Patients arrive with different conditions:

- Someone with a paper cut waits.
    
- Someone having a heart attack is treated immediately.
    

SOC analysts do the same with security alerts.

Example:

```
Alert 1:
Failed login attempt

↓

Maybe someone mistyped their password.

Priority: Low
```

```
Alert 2:
Administrator account added to Domain Admins

↓

Possible privilege escalation.

Priority: Critical
```

The analyst must quickly determine which alerts need immediate attention.

---

# Why Is Triaging Important?

Large organizations generate **thousands or even millions of alerts daily**.

For example:

|Alert|Action|
|---|---|
|User entered wrong password once|Ignore/Low Priority|
|Antivirus updated|Ignore|
|Malware detected|Investigate|
|Domain Admin created|Immediate Response|

Without triaging:

- Analysts waste time on harmless events.
    
- Real attacks might go unnoticed.
    

---

# The Ideal Alert Triaging Process

## Step 1 : Initial Alert Review

The analyst first reads the alert.

Questions include:

- When did it happen?
    
- Which computer?
    
- Which user?
    
- Which IP address?
    
- Which detection rule triggered?
    

Example:

```
Alert:

Event ID: 4625
User: Administrator
Source IP: 192.168.1.25
Host: SERVER01
Time: 10:35 AM
```

This is only the starting point.

---

## Step 2: Analyze the Logs

One alert rarely tells the full story.

The analyst examines related logs.

Example:

```
4625 Failed Login

↓

4625 Failed Login

↓

4625 Failed Login

↓

4624 Successful Login
```

This pattern suggests a possible **brute-force attack**.

---

## Step 3: Classify the Alert

Every organization defines severity levels.

Example:

|Severity|Meaning|
|---|---|
|Informational|No action needed|
|Low|Minor issue|
|Medium|Needs investigation|
|High|Serious concern|
|Critical|Immediate response|

Example:

```
Antivirus update

↓

Informational
```

```
Ransomware detected

↓

Critical
```

---

## Step 4: Correlate Related Events

One alert alone may not indicate an attack.

The analyst looks for related activity.

Example:

```
4625 Failed Login

↓

4688 PowerShell Started

↓

4104 Suspicious PowerShell Script

↓

Network Connection
```

Individually, these may seem benign.

Together, they strongly suggest malicious behavior.

This is **alert correlation**.

---

## Step 5: Enrich the Alert

Enrichment means adding more context.

Instead of seeing:

```
IP:
8.8.8.8
```

The analyst asks:

- Is this IP malicious?
    
- Which country is it from?
    
- Has it been seen before?
    
- Is it listed in threat intelligence feeds?
    

Other enrichment sources include:

- VirusTotal
    
- Malware sandboxes
    
- DNS lookups
    
- WHOIS records
    
- Threat intelligence platforms
    

The goal is to answer:

> "What else do we know about this alert?"

---

## Step 6: Risk Assessment

Now the analyst determines the impact.

Consider these examples:

```
Malware on an intern's laptop
```

versus

```
Malware on the Domain Controller
```

Same malware.

Very different risk.

Questions include:

- Which system is affected?
    
- Is sensitive data involved?
    
- Could the attacker move laterally?
    
- What would happen if no action is taken?
    

---

## Step 7: Contextual Analysis

Not every alert is malicious.

The analyst asks:

> "What else is happening in the environment?"

Example:

```
Alert:
PowerShell executed.
```

Could this be malicious?

Maybe.

But then IT reports:

> "We deployed Windows updates today."

Now it makes sense.

Another example:

```
Many failed logins
```

Possible explanations:

- Password spraying
    
- User forgot their password
    
- Authentication server issue
    

Context helps distinguish malicious activity from normal operations.

---

## Step 8: Incident Response Planning

If the alert appears legitimate, preparation begins.

The analyst documents:

- Timeline
    
- Affected hosts
    
- Users
    
- Indicators of Compromise (IOCs)
    
- Evidence collected
    

Team responsibilities are assigned.

Example:

```
SOC:
Investigate logs

IT:
Isolate machine

Forensics:
Collect memory image

Management:
Coordinate communications
```

---

## Step 9: Consult IT Operations

SOC analysts don't know everything about the environment.

Sometimes they ask IT:

> "Was this expected?"

Example:

```
New administrator account created
```

IT responds:

> "Yes, we onboarded a new employee today."

The alert is legitimate.

Without checking, this could have resulted in unnecessary escalation.

---

## Step 10: Execute the Response

Based on the findings:

If it's benign:

```
Close Alert

↓

False Positive
```

If it's malicious:

```
Isolate Host

↓

Disable User

↓

Block IP

↓

Start Incident Response
```

---

## Step 11: Escalation

Not every alert stays with the Tier 1 SOC analyst.

Escalation means:

> "This is beyond my responsibility."

Common escalation triggers:

- Domain Controller compromised
    
- Ransomware detected
    
- Active attacker
    
- Large-scale malware outbreak
    
- Insider threat
    
- Data theft
    
- Advanced Persistent Threat (APT)
    

The analyst provides:

- Alert summary
    
- Severity
    
- Impact
    
- Evidence
    
- Risk assessment
    
- Recommended next steps
    

The incident is handed to Tier 2, Tier 3, Incident Response, or Management.

---

## Step 12: Continuous Monitoring

Even after escalation, monitoring continues.

The analyst watches for:

- New alerts
    
- Additional compromised systems
    
- Lateral movement
    
- Escalation of attacker activity
    
- Successful containment
    

The incident evolves, and the SOC tracks those changes.

---

## Step 13: De-escalation

Once the incident is contained:

- Systems are restored.
    
- Malware is removed.
    
- Accounts are secured.
    
- The attacker is no longer active.
    

The incident can be downgraded or closed.

Lessons learned are documented to improve future detection and response.

---

# Complete Workflow

```text
Alert Generated
        │
        ▼
Review Alert
        │
        ▼
Analyze Logs
        │
        ▼
Classify Severity
        │
        ▼
Correlate Related Events
        │
        ▼
Enrich Alert
        │
        ▼
Assess Risk
        │
        ▼
Analyze Context
        │
        ▼
Consult IT (if needed)
        │
        ▼
Respond
        │
        ├──────────────► False Positive → Close Alert
        │
        ▼
Escalate (if required)
        │
        ▼
Monitor Incident
        │
        ▼
Contain & Recover
        │
        ▼
De-escalate & Close
```

## CPTS Exam Tips

Focus on understanding the purpose of each stage rather than memorizing the list:

- **Review** → Understand what happened.
    
- **Analyze** → Gather evidence.
    
- **Classify** → Determine priority.
    
- **Correlate** → Connect related events.
    
- **Enrich** → Add external and internal context.
    
- **Assess Risk** → Evaluate business impact.
    
- **Contextual Analysis** → Decide if the activity is expected or suspicious.
    
- **Consult IT** → Rule out legitimate changes or maintenance.
    
- **Respond** → Take appropriate action.
    
- **Escalate** → Involve higher-level responders when necessary.
    
- **Monitor** → Track the incident until it's resolved.
    
- **De-escalate** → Close the incident once the risk has been mitigated.
    



