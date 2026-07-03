
# MITRE ATT&CK 
## What is MITRE ATT&CK?

- **MITRE ATT&CK** = **Adversarial Tactics, Techniques, and Common Knowledge**
    
- A knowledge base of **real-world attacker behaviors (TTPs)**.
    
- Helps defenders understand, detect, and respond to cyberattacks.
    
- Regularly updated by MITRE.
    

---

# Key Terms

### Tactics

- **The attacker's goal ("Why")**
    
- Examples:
    
    - Initial Access
        
    - Execution
        
    - Persistence
        
    - Credential Access
        
    - Lateral Movement
        
    - Exfiltration
        

### Techniques

- **How the attacker achieves the goal ("How")**
    
- Example:
    
    - Phishing
        
    - PowerShell
        
    - Pass-the-Hash
        

### TTPs

- **Tactics + Techniques + Procedures**
    
- Describe how attackers operate in real attacks.
    

---

# ATT&CK Matrix

- Organized into different environments:
    
    - Enterprise
        
    - Mobile
        
    - Cloud
        
- Maps **Tactics → Techniques** used by attackers.
    

---

# MITRE ATT&CK Use Cases

### 1. Detection & Response

- Build SIEM/EDR detection rules.
    
- Improve incident response.
    
- Detect known attacker techniques.
    

### 2. Security Assessment & Gap Analysis

- Identify missing security controls.
    
- Find weaknesses in defenses.
    
- Prioritize security improvements.
    

### 3. SOC Maturity Assessment

- Measure how well the SOC:
    
    - Detects attacks
        
    - Responds to incidents
        
    - Mitigates threats
        

### 4. Threat Intelligence

- Provides a common language for attacker behavior.
    
- Improves information sharing.
    

### 5. Cyber Threat Intelligence (CTI)

- Adds context to:
    
    - TTPs
        
    - Indicators of Compromise (IOCs)
        
    - Potential targets
        
- Helps make better security decisions.
    

### 6. Behavioral Analytics

- Maps attacker behaviors to user/system activity.
    
- Detects anomalies and suspicious behavior.
    

### 7. Red Teaming & Penetration Testing

- Simulates real attacker techniques.
    
- Tests defensive capabilities.
    

### 8. Training & Education

- Excellent resource for learning attacker techniques.
    
- Widely used by SOC analysts and threat hunters.
    

---

# Why MITRE ATT&CK is Important

- Standardizes attacker behavior.
    
- Improves threat detection.
    
- Enhances threat hunting.
    
- Helps build detection rules.
    
- Strengthens incident response.
    
- Improves overall SOC effectiveness.
    

---

# Common ATT&CK Tactics (Examples)

| Tactic               | Goal                        |
| -------------------- | --------------------------- |
| Reconnaissance       | Gather information          |
| Initial Access       | Gain entry                  |
| Execution            | Run malicious code          |
| Persistence          | Maintain access             |
| Privilege Escalation | Gain higher privileges      |
| Defense Evasion      | Avoid detection             |
| Credential Access    | Steal credentials           |
| Discovery            | Learn about the environment |
| Lateral Movement     | Move to other systems       |
| Collection           | Gather sensitive data       |
| Exfiltration         | Steal data                  |
| Impact               | Disrupt or damage systems   |

---

# Exam/Interview Points 

- **MITRE ATT&CK = Knowledge base of attacker TTPs.**
    
- **Tactic = Why** an attacker performs an action.
    
- **Technique = How** the attacker performs it.
    
- **TTP = Tactics + Techniques + Procedures.**
    
- Used for:
    
    - Threat Detection
        
    - Threat Hunting
        
    - Threat Intelligence
        
    - SOC Maturity
        
    - Gap Analysis
        
    - Red Teaming
        
    - Penetration Testing
        
    - Security Training
        
- **ATT&CK Matrix** maps attacker goals (tactics) to methods (techniques).