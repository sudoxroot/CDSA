
# SIEM Use Cases

## What is a SIEM Use Case?

- A **SIEM use case** is a detection rule that identifies specific suspicious activities.
    
- Converts multiple log events into meaningful security alerts.
    
- Helps SOC analysts detect and respond to attacks.
    

**Example:**

- **10 failed logins in 4 minutes** → SIEM correlates events → Generates **Brute Force** alert.
    

---

# SIEM Use Case Development Lifecycle

### 1. Requirements

- Define what needs to be detected.
    
- Example:
    
    - Brute force attack
        
    - Ransomware
        
    - Privilege escalation
        

---

### 2. Data Points

Identify log sources:

- Windows
    
- Linux
    
- Servers
    
- VPN
    
- Applications
    
- Firewalls
    
- Endpoints
    

Logs should include:

- Username
    
- Timestamp
    
- Source IP
    
- Destination
    
- Hostname
    

---

### 3. Log Validation

Verify logs contain all required fields:

- User
    
- Time
    
- Source
    
- Destination
    
- Machine
    
- Application
    

---

### 4. Design & Implementation

Define:

- **Condition** (When to alert)
    
- **Aggregation** (Correlate events)
    
- **Priority** (Severity)
    

Example:

- 10 failed logins within 4 minutes → High severity alert
    

---

### 5. Documentation

Create **SOP (Standard Operating Procedure)** containing:

- Alert conditions
    
- Severity
    
- Escalation steps
    
- Teams to notify
    

---

### 6. Onboarding

- Test in development.
    
- Reduce false positives.
    
- Deploy to production.
    

---

### 7. Fine-Tuning

- Update detection rules.
    
- Whitelist trusted activities.
    
- Improve detection accuracy.
    

---

# How to Build SIEM Use Cases

1. Identify risks.
    
2. Collect relevant logs.
    
3. Map to **MITRE ATT&CK/Kill Chain**.
    
4. Define alert severity.
    
5. Measure:
    
    - **TTD** (Time to Detection)
        
    - **TTR** (Time to Response)
        
6. Create SOP.
    
7. Develop Incident Response Plan (IRP).
    
8. Define SLAs/OLAs.
    
9. Audit and improve rules.
    
10. Maintain documentation.
    

---

# Important Terms

### TTD (Time to Detection)

- Time taken to detect an attack.
    

### TTR (Time to Response)

- Time taken to respond after detection.
    

### SOP

- Step-by-step analyst instructions for handling alerts.
    

### IRP (Incident Response Plan)

- Process for handling confirmed incidents.
    

### SLA

- Service Level Agreement between teams.
    

### OLA

- Operational Level Agreement between internal teams.
    

---

# Example 1: MSBuild Started by Microsoft Office

### Risk

- **MSBuild.exe** launched by:
    
    - Word
        
    - Excel
        

Normal?

- ❌ Usually No
    

Possible Attack

- Living-off-the-Land Binary (LoLBin)
    
- Malicious script execution
    

### Severity

- **HIGH**
    

### MITRE ATT&CK Mapping

|Tactic|Technique|
|---|---|
|TA0002|Execution|
|TA0005|Defense Evasion|
|T1127|Trusted Developer Utilities Proxy Execution|
|T1127.001|MSBuild|

### SOP Investigation

Check:

- `process.name`
    
- `process.parent.name`
    
- `event.action`
    
- Hostname
    
- Username
    
- User activity (±2 days)
    
- AV logs
    
- Proxy logs
    
- System logs
    

### Fine-Tuning

- Exclude legitimate developer machines.
    
- Whitelist expected MSBuild activity.
    

---

# Example 2: MSBuild Making Network Connections

### Risk

- **MSBuild.exe** connects to external IPs.
    

Possible Attack

- Malware communicating with C2 server.
    
- Living-off-the-Land abuse.
    

### Severity

- **MEDIUM**
    

Reason:

- MSBuild may legitimately contact Microsoft servers.
    
- Higher chance of false positives.
    

### MITRE Mapping

|Tactic|Technique|
|---|---|
|TA0002|Execution|

### SOP Investigation

Check:

- `event.action`
    
- Destination IP
    
- IP reputation
    
- Threat Intelligence
    
- Network logs
    

---

# Why SIEM Use Cases Matter

- Detect attacks automatically.
    
- Reduce analyst workload.
    
- Correlate multiple events.
    
- Improve incident response.
    
- Reduce false positives through tuning.
    

---

# Exam/Interview Points 

- **SIEM Use Case = Detection rule for specific attack scenarios.**
    
- **Lifecycle:** Requirements → Data Points → Log Validation → Design → Documentation → Onboarding → Testing → Fine-Tuning.
    
- **TTD:** Time to detect an attack.
    
- **TTR:** Time to respond to an attack.
    
- **SOP:** Analyst playbook.
    
- **IRP:** Incident handling process.
    
- **MSBuild started by Word/Excel** → **High Severity**, **MITRE T1127.001 (MSBuild)**.
    
- **MSBuild making outbound connections** → **Medium Severity** (possible legitimate traffic).
    
- **Fine-tuning** (whitelisting and rule optimization) reduces false positives.