
## What is a SOC?

- A **Security Operations Center (SOC)** is a team that **continuously monitors, detects, investigates, and responds** to cybersecurity threats.
    
- Uses people, processes, and security tools to protect an organization.
    
- Works closely with the **Incident Response (IR)** team.
    

---

# Main Responsibilities of a SOC

- Monitor security events 24/7
    
- Detect and investigate threats
    
- Respond to security incidents
    
- Perform threat hunting
    
- Use threat intelligence
    
- Improve overall security posture
    

---

# Security Tools Used by SOC

- **SIEM** : Centralized log collection, correlation, and alerting
    
- **IDS/IPS** : Detects/blocks network attacks
    
- **EDR** : Monitors and responds to endpoint threats
    
- Threat Intelligence Platforms
    
- Malware Analysis & Digital Forensics tools
    

---

# Incident Response Process

1. **Detection**
    
2. **Triage** (Prioritize)
    
3. **Containment**
    
4. **Eradication** (Remove threat)
    
5. **Recovery**
    
6. **Lessons Learned**
    

---

# SOC Roles

| Role                               | Responsibility                                             |
| ---------------------------------- | ---------------------------------------------------------- |
| **SOC Director**                   | Strategy, budgeting, overall leadership                    |
| **SOC Manager**                    | Daily SOC operations                                       |
| **Tier 1 Analyst**                 | Monitor alerts, initial triage, escalate incidents         |
| **Tier 2 Analyst**                 | Investigate alerts, reduce false positives, mitigation     |
| **Tier 3 Analyst**                 | Advanced investigations, threat hunting, complex incidents |
| **Detection Engineer**             | Create and improve SIEM/EDR/IDS detection rules            |
| **Incident Responder**             | Containment, forensics, remediation                        |
| **Threat Intelligence Analyst**    | Research attacker TTPs and threats                         |
| **Security Engineer**              | Deploy and maintain security infrastructure                |
| **Compliance Specialist**          | Ensure regulatory compliance (ISO, PCI DSS, HIPAA, GDPR)   |
| **Security Awareness Coordinator** | Employee cybersecurity training                            |

---

# SOC Analyst Tiers

### Tier 1 (First Responder)

- Monitor SIEM alerts
    
- Validate alerts
    
- Perform initial investigation
    
- Escalate serious incidents
    

### Tier 2

- Deep investigation
    
- Analyze attack patterns
    
- Develop mitigation
    
- Tune SIEM to reduce false positives
    

### Tier 3

- Expert analysts
    
- Handle advanced threats
    
- Threat hunting
    
- Improve detections
    
- Mentor lower tiers
    

---

# SOC Stages

## SOC 1.0

- Focused mainly on **network/perimeter security**
    
- Separate security tools
    
- Poor integration
    
- Many uncorrelated alerts
    
- Reactive approach
    

---

## SOC 2.0

- Intelligence-driven SOC
    
- Integrates:
    
    - SIEM
        
    - Threat Intelligence
        
    - Network Flow Analysis
        
    - Layer-7 Analysis
        
- Detects advanced threats (APT, botnets, low-and-slow attacks)
    
- Includes:
    
    - Vulnerability Management
        
    - Configuration Management
        
    - Incident Response
        
    - Digital Forensics
        

---

## Cognitive SOC (Next Generation)

- Uses **AI/Machine Learning**
    
- Learns from previous incidents
    
- Improves detection over time
    
- Better collaboration between security and business teams
    
- Standardized incident response
    

---

# Key Concepts

### Threat Hunting

- Proactively searches for hidden threats before alerts are generated.
    

### Threat Intelligence

- Information about attackers, malware, IOCs, TTPs, and emerging threats.
    

### Digital Forensics

- Investigates attacks and identifies the root cause.
    

### Security Posture

- Overall strength of an organization's cybersecurity defenses.
    

---

# Exam/Interview Points 

- **SOC = 24/7 security monitoring and incident response.**
    
- **Primary Goal:** Detect → Investigate → Respond → Recover.
    
- **SOC uses:** SIEM, IDS/IPS, EDR, Threat Intelligence.
    
- **Tier 1:** Monitor & triage.
    
- **Tier 2:** Investigate & mitigate.
    
- **Tier 3:** Threat hunting & advanced analysis.
    
- **Detection Engineers** create detection rules.
    
- **Incident Responders** perform containment, remediation, and forensics.
    
- **SOC Evolution:** **SOC 1.0 → SOC 2.0 → Cognitive SOC (AI-driven)**.


  

## Question 1

### True or false? SOC 2.0 follows a proactive defense approach.

### Solution:

- As They rely heavily on continuous monitoring, AI, and automation to hunt and neutralize threats before they escalate into breaches, it would be `ture`.