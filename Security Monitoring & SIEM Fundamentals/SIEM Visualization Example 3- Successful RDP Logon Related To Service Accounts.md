
# SIEM Visualization 

## Goal

Monitor **successful RDP logons** performed using **service accounts**.

> **Why?** Service accounts should **never** be used for RDP logins in a corporate environment. Such activity may indicate **credential misuse or compromise**.

**Assumption:**

- All service accounts start with:
    

```text
svc-
```

---

# Required Filters

### Successful Login

```kql
event.code:4624
```

### RDP (Remote Interactive) Logon

```kql
winlog.logon.type:RemoteInteractive
```

### Service Accounts

```kql
user.name:svc-*
```

---

# Complete KQL Query

```kql
user.name:svc-* AND event.code:4624 AND winlog.logon.type:RemoteInteractive
```

---

# Dashboard Configuration

### Index Pattern

```text
windows*
```

### Visualization Type

- **Table**
    

---

# Rows Configuration

### 1. Service Account

Field:

```text
user.name.keyword
```

---

### 2. Target Machine

Field:

```text
host.hostname.keyword
```

Displays the machine where the RDP login occurred.

---

### 3. Source IP

Field:

```text
related.ip.keyword
```

Displays the IP address that initiated the RDP session.

---

# Metrics

Use:

- **Count**
    

Shows the number of successful RDP logins.

---

# Final Table Columns

| Column              | Description                     |
| ------------------- | ------------------------------- |
| **Service Account** | Account used for RDP            |
| **Target Host**     | Machine receiving the RDP login |
| **Source IP**       | IP initiating the RDP session   |
| **# of Logins**     | Number of successful RDP logins |

---

# Important Windows Values

| Field               | Value                 | Meaning                   |
| ------------------- | --------------------- | ------------------------- |
| `event.code`        | **4624**              | Successful logon          |
| `winlog.logon.type` | **RemoteInteractive** | RDP login (Logon Type 10) |

---

# Why Monitor Service Accounts?

- Service accounts usually have **high privileges**.
    
- They are intended for **services and applications**, **not interactive logins**.
    
- RDP logins using service accounts may indicate:
    
    - Credential theft
        
    - Privilege abuse
        
    - Lateral movement
        
    - Malicious administrator activity
        

---

# Important Notes

- Use **`.keyword`** fields for **visualization rows** (aggregation).
    
- Use **non-`.keyword`** fields in **KQL queries**.
    

Example:

- Visualization field:
    
    ```text
    user.name.keyword
    ```
    
- KQL query:
    
    ```kql
    user.name:svc-*
    ```
    

---

# Exam/Interview Points 

- **Event ID 4624** = Successful Windows logon.
    
- **Logon Type `RemoteInteractive` (Type 10)** = RDP login.
    
- Service accounts should **not** perform RDP logins.
    
- Filter service accounts using:
    
    ```kql
    user.name:svc-*
    ```
    
- **Index Pattern:** `windows*`
    
- **Visualization:** Table
    
- **Rows:** `user.name.keyword`, `host.hostname.keyword`, `related.ip.keyword`
    
- **Metric:** `Count`
    
- Investigate any successful RDP login by a service account as it is **highly suspicious**.


  

## Question 1

### Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Dashboard". Browse the visualization we created or the "RDP logon for service account" visualization, if it is available, and enter the IP of the machine that initiated the successful RDP logon using service account credentials as your answer.
## Solution :

- After navigating we came to see that the "RDP logon for service account" visualization is already created,
- So after seeing the data, we can easily detect IP of the machine that initiated the successful RDP logon.















