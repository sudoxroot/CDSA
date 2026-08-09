
## What are Splunk Apps?

- **Splunk Apps** are pre-built packages that extend the functionality of **Splunk Enterprise** or **Splunk Cloud**.
    
- They are designed for **specific technologies or use cases**.
    
- Think of them as **plugins with ready-made dashboards, searches, reports, and configurations**.
    

### Easy Analogy

- **Splunk = Android phone**
    
- **Splunk Apps = Apps from Play Store**
    
- Install an app to add new features without building everything yourself.
    

---

# What Can Splunk Apps Provide?

A Splunk app can include:

- 📥 Custom data inputs
    
- 📊 Dashboards
    
- 📈 Visualizations
    
- 🚨 Alerts
    
- 📄 Reports
    
- 🔍 Saved searches
    
- ⚙️ Search macros
    
- 🧠 Knowledge objects
    

---

# Why Use Splunk Apps?

Instead of creating dashboards and searches from scratch, apps provide everything already configured.

### Benefits

- Faster deployment
    
- Easier analysis
    
- Standardized dashboards
    
- Better visibility
    
- Multiple workspaces for different teams
    

Example:

- SOC Team → Security App
    
- IT Team → Infrastructure App
    
- Network Team → Network Monitoring App
    

All can use the **same Splunk instance**.

---

# Where to Download Apps?

Apps are available on **Splunkbase**.

Think of **Splunkbase** as:

> **App Store for Splunk**

---

# Splunk Apps for SIEM

Security-focused Splunk apps help analysts:

- Detect attacks
    
- Investigate incidents
    
- Visualize security events
    
- Hunt threats
    
- Respond to incidents
    

These apps ingest and analyze logs from:

- Windows
    
- Linux
    
- Sysmon
    
- Firewalls
    
- IDS/IPS
    
- Cloud services
    
- Endpoint security tools
    

---

# Things to Consider Before Installing Apps

Some apps consume many resources.

Always consider:

### 1. Data Volume

More logs = more storage and processing.

### 2. Hardware

Need sufficient:

- CPU
    
- RAM
    
- Disk
    

### 3. Licensing

Premium apps may require additional licenses.

### 4. License Usage

If the app ingests more data, your daily license usage also increases.

---

# Sysmon App for Splunk

In this module, the app used is:

**Sysmon App for Splunk** (developed by **Mike Haag**)

Purpose:

- Analyze **Sysmon logs**
    
- Visualize Windows activity
    
- Help detect suspicious behavior
    

---

# Installation Steps

### Step 1

Create a free account on **Splunkbase**.

### Step 2

Download the **Sysmon App for Splunk**.

### Step 3

In Splunk:

**Apps → Manage Apps → Install app from file**

Upload:

```
sysmon-app-for-splunk_200.tgz
```

---

# Configure the Search Macro

The app expects Sysmon events under a specific index and sourcetype.

Original macro:

```spl
sourcetype="XmlWinEventLog:Microsoft-Windows-Sysmon/Operational"
```

Changed to:

```spl
index="main" sourcetype="WinEventLog:Sysmon"
```

### Why?

Because the lab stores Sysmon logs as:

- **Index:** `main`
    
- **Sourcetype:** `WinEventLog:Sysmon`
    

If the macro doesn't match your environment, the dashboards won't display data.

---

# Viewing the Dashboard

Open:

```
Apps
   ↓
Sysmon App for Splunk
   ↓
File Activity
```

Set the time range to:

```
All Time
```

Click:

```
Submit
```

---

# Common Dashboard Issue

Problem:

```
Top Systems
No results found
```

### Cause

The search uses:

```spl
top Computer
```

But **Sysmon Event ID 11** doesn't have a field called:

```
Computer
```

It contains:

```
ComputerName
```

---

# Fix

Edit the search:

Replace:

```spl
top Computer
```

with

```spl
top ComputerName
```

Example:

```spl
sysmon EventCode=11
| top ComputerName
```

After applying the change, the dashboard populates correctly.

---

# Event ID 11

Sysmon Event ID:

```
11
```

Represents:

**File Creation**

Useful for detecting:

- Malware dropping files
    
- Payload creation
    
- Suspicious executables
    
- Scripts written to disk
    

---

# Important Lesson

If a dashboard shows **No Results**:

1. Check the index.
    
2. Check the sourcetype.
    
3. Verify field names.
    
4. Modify the search to match your data.
    

Dashboards are not universal—they often require customization for your environment.

---

# Exam Tips ⭐

- **Splunk Apps** = Pre-built packages that extend Splunk functionality.
    
- **Splunkbase** = Official repository for Splunk apps.
    
- Apps can include dashboards, alerts, reports, searches, macros, and visualizations.
    
- Security apps help with threat detection, investigation, and response.
    
- Consider hardware, data volume, and licensing before installing apps.
    
- **Sysmon App for Splunk** is used to analyze Sysmon logs.
    
- Configure the search macro to match your environment's **index** and **sourcetype**.
    
- **Sysmon Event ID 11 = File Creation**.
    
- If a dashboard returns no results, verify the search uses the correct field names (e.g., `ComputerName` instead of `Computer`).




-   
    
    ## Question 1
    
    ### Access the Sysmon App for Splunk and go to the "Reports" tab. Fix the search associated with the "Net - net view" report and provide the complete executed command as your answer. Answer format: net view /Domain:_.local
    
    - ## Solution:
    - So it was a simple question, I opened the given tab and then search for Net-net view, got the targetted thing.
    - I opened it and then searched 
    - ```
      "*net view/Domain:*"
      ```
    - There I got the answer
- ## Question 2
    
    ### Access the Sysmon App for Splunk, go to the "Network Activity" tab, and choose "Network Connections". Fix the search and provide the number of connections that SharpHound.exe has initiated as your answer.
    - I opened the given tab, hover over the tab and clicked on the search icon
    - a new tab is opened, here I search
    - ```
      EventCode=3 AND Message="*SharpHound.exe*"
      ```
    - eventid 3 because of the network connection initiated and sharphound.exe was already given
    - there was total hits of number 6, so it was the right answer






