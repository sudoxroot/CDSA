
# SIEM (Security Information and Event Management) – Short Notes

## What is SIEM?

- **SIEM = SIM + SEM**
    
    - **SIM (Security Information Management):** Log collection, storage, reporting.
        
    - **SEM (Security Event Management):** Real-time event monitoring, correlation, and alerts.
        
- Collects, analyzes, and correlates logs from multiple sources.
    
- Detects threats, generates alerts, and helps incident response.
    
- Central platform for monitoring an organization's security.
    

---

# Evolution of SIEM

- **2005:** Gartner introduced SIEM.
    
- Combines:
    
    - **SIM:** Long-term log management.
        
    - **SEM:** Real-time event detection and alerting.
        
- Provides both historical analysis and real-time threat detection.
    

---

# How SIEM Works

1. Collect logs from:
    
    - Endpoints
        
    - Servers
        
    - Firewalls
        
    - IDS/IPS
        
    - Databases
        
    - Applications
        
2. Normalize data into a common format.
    
3. Correlate events from different sources.
    
4. Detect suspicious activity.
    
5. Generate alerts for SOC analysts.
    
6. Analysts investigate and respond.
    

---

# SIEM Alerts

- Notify SOC teams about possible attacks.
    
- Can be sent via:
    
    - Email
    - Console pop-ups
    - SMS        
    - Phone notifications
        
- SIEM generates **many alerts**, so tuning is essential to reduce noise and false positives.
    

---

# Why SIEM is Better than IDS/IPS

- **IDS/IPS:** Detects activity on individual devices.    
- **SIEM:** Combines logs from many devices to detect larger attack patterns.
- SIEM complements—not replaces—IDS/IPS.
    

---

# SIEM Business Requirements

### 1. Log Aggregation & Normalization

- Centralizes logs from all systems.
    
- Converts different log formats into one standard format.
    
- Improves visibility across the network.
    

### 2. Threat Alerting

- Detects suspicious behavior.
    
- Sends real-time alerts.
    
- Uses analytics and threat intelligence.
    

### 3. Contextualization & Response

- Adds context to alerts:
    
    - Who?
        
    - What?
        
    - When?
        
    - Where?
        
- Reduces false positives.
    
- Supports automated responses.
    

### 4. Compliance

Helps meet regulations like:

- PCI DSS
    
- HIPAA
    
- GDPR
    
- ISO
    

Provides:

- Audit logs
    
- Compliance reports
    
- Log retention
    

---

# Data Flow in SIEM

**Data Sources → Data Ingestion → Normalization/Aggregation → Correlation Engine → Detection Rules → Dashboards/Alerts/Incidents**

---

# Benefits of SIEM

- Centralized log management.
    
- Faster threat detection.
    
- Faster incident response.
    
- Correlates events from multiple sources.
    
- Detects attack patterns.
    
- Provides dashboards and reports.
    
- Supports compliance requirements.
    
- Modern SIEMs use **AI/behavior analytics**.
    
- Reduces financial and reputational damage from attacks.
    

---

# Exam/Interview Points 

- **SIEM = SIM + SEM**
    
- **Main Functions:** Collect → Normalize → Correlate → Detect → Alert → Respond
    
- **Core Features:**
    
    - Log aggregation
        
    - Event correlation
        
    - Real-time alerting
        
    - Dashboards
        
    - Reporting
        
    - Compliance
        
- **SIEM does NOT replace IDS/IPS** it works alongside them.
    
- **SOC analysts** primarily use SIEM for monitoring, investigation, and incident response.