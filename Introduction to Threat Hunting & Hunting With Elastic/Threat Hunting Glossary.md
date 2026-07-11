
# Cyber Threat Hunting Notes (Simple & Easy)

---

# 1. Adversary

### Definition

An **adversary** is anyone trying to attack your organization without permission.

Think of them as **the enemy**.

### Goal

- Steal money
    
- Steal data
    
- Spy on organizations
    
- Damage systems
    
- Disrupt operations
    

### Types

|Type|Goal|
|---|---|
|Cyber Criminal|Money|
|Insider Threat|Abuse internal access|
|Hacktivist|Political/Social reasons|
|State-sponsored|Espionage, military, intelligence|

### Example

A ransomware gang encrypts a hospital's files.

The ransomware gang = **Adversary**

---

# 2. Advanced Persistent Threat (APT)

APT = **Highly organized attackers** with lots of resources.

Usually:

- Nation states
    
- Military intelligence
    
- Government-backed groups
    

They attack:

- Governments
    
- Banks
    
- Hospitals
    
- Defense companies
    

---

## Why "Advanced"?

Not necessarily because their malware is advanced.

Advanced means they have:

- Good planning
    
- Skilled people
    
- Large budget
    
- Patience
    

---

## Why "Persistent"?

Because they stay inside networks for **weeks, months, or even years**.

They don't attack once and leave.

---

### Example

A Chinese espionage group remains inside a defense contractor's network for 9 months stealing documents.

That is an **APT**.

---

# 3. TTPs (Tactics, Techniques and Procedures)

One of the MOST important CPTS topics.

Think:

> **Why → How → Exact Steps**

---

## Tactics

**Why are they doing it?**

High-level objective.

Examples

- Initial Access
    
- Credential Access
    
- Persistence
    
- Exfiltration
    

Example

"I want administrator credentials."

That objective is the **tactic**.

---

## Techniques

**How will they accomplish it?**

General method.

Examples

- Phishing
    
- Password spraying
    
- Exploiting SMB
    
- DLL Injection
    

Example

Use phishing emails.

That is the **technique**.

---

## Procedures

**Exactly how do they perform it?**

Specific steps.

Example

1. Create fake Microsoft login page
    
2. Send phishing email
    
3. Victim enters password
    
4. Store credentials
    
5. Log in
    

These steps are the **procedure**.

---

## Easy Memory

```
Tactic
↓
Why?

Technique
↓
How?

Procedure
↓
Exactly how?
```

---

# 4. Indicator

Indicator =

> Data + Context

Raw data alone isn't useful.

Example

```
185.88.22.10
```

Just an IP.

No meaning.

Now add context:

```
185.88.22.10
Known C2 server
Used by ransomware group
Seen yesterday
```

Now it becomes an **Indicator**.

---

# 5. Threat

Threat depends on **three things**.

```
Intent
+
Capability
+
Opportunity
=
Threat
```

---

## Intent

Why does the attacker want to attack?

Examples

- Money
    
- Espionage
    
- Revenge
    

---

## Capability

Can they actually perform the attack?

Do they have:

- Malware?
    
- Skills?
    
- Budget?
    
- Infrastructure?
    

---

## Opportunity

Can they attack right now?

Examples

- Vulnerability exists
    
- Password leaked
    
- Employee clicks phishing email
    

---

### Easy Example

A ransomware gang:

Intent

✔ Wants money

Capability

✔ Has ransomware

Opportunity

✔ Company exposed RDP server

Result:

Threat exists.

---

# 6. Campaign

Campaign = Multiple related attacks.

They usually share:

- Same malware
    
- Same TTPs
    
- Same goals
    

Think:

One attacker conducting many attacks.

---

Example

An attacker sends phishing emails to 40 companies using the same malware.

That entire operation is a **campaign**.

---

# 7. Indicators of Compromise (IOCs)

IOCs = Evidence that an attack happened.

Examples

- Malicious hash
    
- Bad IP
    
- Bad domain
    
- Registry key
    
- Malware filename
    
- Suspicious process
    

Think:

IOCs = Crime scene evidence.

---

# 8. Pyramid of Pain

One of the most important concepts in threat hunting.

It asks:

> "How painful is it for attackers if we detect this?"

Higher = More painful.

```
          TTPs
          Tools
 Network/Host Artifacts
      Domain Names
      IP Addresses
      Hashes
```

---

## 1. Hashes (Trivial)

Very easy for attackers to change.

Example

Change one byte.

New hash.

Done.

---

## 2. IP Addresses (Easy)

Attackers change

- VPN
    
- Proxy
    
- VPS
    
- TOR
    

Very simple.

---

## 3. Domains (Simple)

Register another domain.

Example

Old

```
microsoft-login.com
```

New

```
secure-office365.net
```

Easy.

---

## 4. Network/Host Artifacts (Annoying)

These are traces attackers leave behind.

Examples

Network

- DNS requests
    
- SMB traffic
    
- Packet captures
    
- NetFlow
    

Host

- Registry changes
    
- Scheduled tasks
    
- DLL loading
    
- Running processes
    
- Event logs
    

Harder to change.

---

## 5. Tools (Challenging)

Now you're detecting the malware itself.

Examples

- Cobalt Strike
    
- Mimikatz
    
- PowerShell scripts
    

Changing tools requires effort.

---

## 6. TTPs (Most Painful)

Very difficult to change.

Changing TTPs means attackers must change how they operate.

Example

If you detect:

- Spear phishing
    
- Credential dumping
    
- Lateral movement
    
- Persistence methods
    

The attacker must redesign the attack.

This is why threat hunters focus heavily on **TTPs** rather than just IPs or hashes.

---

# Pyramid of Pain Memory Trick

|Indicator|Pain to attacker|
|---|---|
|Hash|Very Low|
|IP|Low|
|Domain|Medium|
|Artifacts|High|
|Tools|Very High|
|TTPs|Extremely High|

Higher = Better detection.

---

# 9. Hash Values

Hash = Digital fingerprint.

Examples

- MD5
    
- SHA1
    
- SHA256
    

Used to identify malware.

Example

```
malware.exe

↓

SHA256

↓

A8D7C...
```

Change one byte.

↓

Completely different hash.

---

# 10. IP Address

Attackers use IPs for:

- Command and Control (C2)
    
- Malware hosting
    
- Phishing
    

Problem:

IPs change easily.

---

# 11. Domain Names

Examples

```
evil-update.com
login-office365.net
```

Attackers frequently register new domains.

---

# 12. Network Artifacts

Found inside network logs.

Examples

- DNS logs
    
- Firewall logs
    
- Proxy logs
    
- Packet captures
    
- NetFlow
    

Example

```
Computer contacts
abc-malware.com
every 5 minutes.
```

That repeated communication is a **network artifact**.

---

# 13. Host Artifacts

Found on the endpoint itself.

Examples

- Registry keys
    
- DLLs
    
- Scheduled tasks
    
- Event logs
    
- Running processes
    
- Services
    

Example

```
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
```

Malware adds itself here.

That registry entry is a **host artifact**.

---

# 14. Tools

Software attackers use.

Examples

- Malware
    
- Exploit kits
    
- Scripts
    
- Credential dumpers
    
- Remote access tools
    

Examples

- Mimikatz
    
- Cobalt Strike
    

---

# 15. Diamond Model

The Diamond Model helps analyze an intrusion by focusing on **four connected components**.

```
          Adversary
           /      \
Capability          Infrastructure
           \      /
             Victim
```

---

## 1. Adversary

Who attacked?

Examples

- Ransomware group
    
- Nation-state
    
- Insider
    

---

## 2. Capability

What did they use?

Examples

- Malware
    
- Phishing
    
- Exploits
    
- TTPs
    

---

## 3. Infrastructure

What resources supported the attack?

Examples

- Domains
    
- IPs
    
- Servers
    
- Botnets
    
- C2 servers
    

---

## 4. Victim

Who got attacked?

Examples

- Employee
    
- Company
    
- Government
    
- Hospital
    

---

## Example

Imagine:

- **Adversary:** A ransomware gang.
    
- **Capability:** A phishing email carrying malware.
    
- **Infrastructure:** A malicious domain and C2 server.
    
- **Victim:** A company's finance department.
    

The Diamond Model links these four pieces so analysts can understand the intrusion as a whole, discover related attacks, and improve future detection.

---

# Diamond Model vs. Cyber Kill Chain

|Feature|Diamond Model|Cyber Kill Chain|
|---|---|---|
|Focus|Who and what is involved in the attack|Stages of the attack|
|Main Elements|Adversary, Capability, Infrastructure, Victim|Reconnaissance → Weaponization → Delivery → Exploitation → Installation → C2 → Actions on Objectives|
|Best Use|Threat intelligence and threat hunting|Detecting or stopping attacks at each phase|

---

# Quick Revision Sheet

|Concept|Remember This|
|---|---|
|Adversary|The attacker|
|APT|Well-funded, persistent attacker (often nation-state)|
|Tactic|**Why** they attack|
|Technique|**How** they attack|
|Procedure|**Exact steps** they follow|
|Indicator|Data + Context|
|Threat|Intent + Capability + Opportunity|
|Campaign|Multiple related attacks with shared TTPs|
|IOC|Evidence of compromise (hashes, IPs, domains, registry keys, etc.)|
|Pyramid of Pain|Higher indicators are harder for attackers to change and more valuable for defenders|
|Network Artifact|Evidence in network traffic/logs|
|Host Artifact|Evidence on an endpoint|
|Tools|Software used by attackers|
|Diamond Model|Adversary + Capability + Infrastructure + Victim|

## Exam Tips

- **Know the difference between Tactics, Techniques, and Procedures**—this is tested frequently.
    
- **Understand why TTPs are at the top of the Pyramid of Pain**: they're expensive and disruptive for attackers to change.
    
- **Don't confuse IOCs with TTPs**: IOCs are artifacts left behind; TTPs describe attacker behavior.
    
- **Remember the Threat formula**: **Intent + Capability + Opportunity = Threat**.
    
- **For the Diamond Model**, always ask four questions: **Who attacked? What did they use? What infrastructure supported them? Who was the victim?**