# SIEM Visualization 
## Goal

Create a dashboard visualization to monitor **failed login attempts against disabled user accounts**.

> A disabled account **cannot log in**, even with the correct password. Windows records this with **SubStatus `0xC0000072`**.

---

# Required Filters

### Failed Login Event

```kql
event.code:4625
```

### Disabled Account

```kql
winlog.event_data.SubStatus:0xC0000072
```

**Combined Filter:**

```kql
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072
```

---

# Dashboard Configuration

### Index Pattern

```text
windows*
```

---

### Visualization Type

- **Table**
    

---

# Rows Configuration

### 1. Disabled Username

Field:

```text
user.name.keyword
```

Settings:

- Top 1000 values
    
- Rank by **Count of records**
    
- Descending order
    

---

### 2. Hostname

Field:

```text
host.hostname.keyword
```

Displays the machine where the failed login occurred.

---

# Metrics

Use:

- **Count**
    

Displays the number of failed login events.

---

# Final Table Columns

|Column|Description|
|---|---|
|**Username**|Disabled account targeted|
|**Hostname**|Machine generating the event|
|**Count**|Number of failed login attempts|

---

# Why This Visualization is Useful

- Detects attacks against **disabled accounts**.
    
- May indicate:
    
    - Credential stuffing
        
    - Brute-force attacks
        
    - Attackers using old/stolen credentials
        
    - Reconnaissance against inactive users
        

---

# Key Windows Values

|Field|Value|Meaning|
|---|---|---|
|`event.code`|**4625**|Failed login|
|`winlog.event_data.SubStatus`|**0xC0000072**|Account is disabled|

---

# Exam/Interview Points ⭐

- **Event ID 4625** = Failed Windows logon.
    
- **SubStatus `0xC0000072`** = Login failed because the **account is disabled**.
    
- **Visualization Type:** Table.
    
- **Index Pattern:** `windows*`.
    
- **Rows:** `user.name.keyword`, `host.hostname.keyword`.
    
- **Metric:** `Count`.
    
- **Purpose:** Identify repeated authentication attempts against disabled accounts, which can indicate malicious activity.



-   
    
    ## Question 1
   
    ### Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Dashboard". Either create a new visualization or edit the "Failed logon attempts [Disabled user]" visualization, if it is available, so that it includes failed logon attempt data related to disabled users including the logon type. What is the logon type in the returned document?
    - when I navigated to the dashboard, I got prebuild visualization for failed logon attempts for Disabled user.
    - I opened it, there I saw nothing like logon type, 
    - I went to gear/edit icon -> Added a new row -> put the filter of logon type(winlog.logon.type.keyword)
    - and got the answer
![[4.png]]

    
- ## Question 2
    
  
    ### Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Dashboard". Either create a new visualization or edit the "Failed logon attempts [Admin users only]" visualization, if it is available, so that it includes failed logon attempt data where the username field contains the keyword "admin" anywhere within it. What should you specify after user.name: in the KQL query?
    ## Solution:
    - as we wanna "admin" anywhere within it, so we can use wildcards (on both sides)
    

