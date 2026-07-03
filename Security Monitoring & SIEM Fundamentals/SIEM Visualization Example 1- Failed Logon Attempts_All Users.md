
# SIEM Dashboards & Visualizations 

## What is a SIEM Dashboard?

- A **dashboard** is a collection of visualizations.
    
- Used to monitor and analyze security events.
    
- Displays data using:
    
    - Tables
        
    - Charts
        
    - Graphs
        
    - Metrics
        

---

# Steps to Create a Dashboard

1. Create a **new dashboard**.
    
2. Create a **visualization**.
    
3. Select **time range** (e.g., Last 15 years).
    
4. Select **index pattern** (e.g., `windows*`).
    
5. Apply filters.
    
6. Choose visualization type.
    
7. Save visualization.
    
8. Save dashboard.
    

---

# Key Dashboard Components

### 1. Time Picker

- Defines the time range for data analysis.
    

Example:

```
Last 15 years
```

---

### 2. Filters

Used to show only relevant events.

Example:

```kql
event.code:4625
```

Shows **failed Windows logon** events.

---

### 3. Index Pattern

Specifies which dataset to query.

Example:

```
windows*
```

---

### 4. Field Search

- Search available fields.
    
- Confirms field exists before using it.
    

Example:

```
user.name.keyword
```

---

### 5. Visualization Types

Common types:

- Table
    
- Bar Chart
    
- Line Chart
    
- Pie Chart
    
- Metric
    

---

# Why Use `.keyword` Fields?

Use **`.keyword`** for:

- Aggregations
    
- Sorting
    
- Grouping
    
- Top values
    

Example:

```
user.name.keyword
```

instead of

```
user.name
```

---

# Building the Table Visualization

### Rows

- `user.name.keyword`
    
- `host.hostname.keyword`
    

### Metrics

- **Count**
    

Displays:

- Username
    
- Hostname
    
- Number of logins
    

---

# Visualization Refinements

## Improve Column Names

Rename fields:

| Original              | New Name        |
| --------------------- | --------------- |
| user.name.keyword     | Username        |
| host.hostname.keyword | Event logged by |
| Count                 | # of logins     |

---

## Add Logon Type

Field:

```
winlog.logon.type.keyword
```

---

## Sort Results

Sort by:

- Count of records
    
- Descending order
    

---

# Excluding Specific Users

Exclude unwanted usernames using filters.

Example:

```
user.name.keyword is not DESKTOP-DPOESND
```

---

# Excluding Computer Accounts

KQL:

```kql
NOT user.name: *$ AND winlog.channel.keyword: Security
```

### Meaning

- `NOT user.name: *$`
    
    - Excludes computer accounts (computer names end with `$`)
        
- `winlog.channel.keyword: Security`
    
    - Only Security logs
        

---

# Final Dashboard Columns

| Column          | Description                 |
| --------------- | --------------------------- |
| Username        | User attempting login       |
| Event logged by | Hostname generating the log |
| Logon Type      | Type of Windows login       |
| # of logins     | Number of events            |

---

# Important KQL Queries

### Failed Login Events

```kql
event.code:4625
```

---

### Exclude Computer Accounts

```kql
NOT user.name: *$ AND winlog.channel.keyword: Security
```

---

# Best Practices

- Use meaningful visualization titles.
    
- Use clear column names.
    
- Filter unnecessary data.
    
- Exclude machine accounts.
    
- Sort results by importance.
    
- Save dashboards after changes.
    

---

# Exam/Interview Points ⭐

- **Dashboard** = Collection of visualizations.
    
- **Visualization Types:** Table, Bar, Pie, Line, Metric.
    
- **Index Pattern:** `windows*`
    
- **Failed Login Event ID:** `4625`
    
- Use **`.keyword`** fields for **aggregation, sorting, and grouping**.
    
- **Metric = Count** to show number of events.
    
- Exclude machine accounts with:
    
    ```kql
    NOT user.name: *$ AND winlog.channel.keyword: Security
    ```
    
- A good SOC dashboard should display:
    
    - Username
        
    - Hostname
        
    - Logon Type
        
    - Event Count

  

## Question 1

### Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Dashboard". Browse the refined visualization we created or the "Failed logon attempts [All users]" visualization, if it is available, and enter the number of logins for the sql-svc1 account as your answer.

### Solution:
- When we'll click on Dashboard, we'll see a visualization is already created with name Failed logon attempts(All users).
- We'll navigate it and will get the counts of logins for sql-svc1.
















