
# Elastic Stack – Short Notes

## What is Elastic Stack?

- Open-source platform for **searching, analyzing, and visualizing logs**.
    
- Commonly used as a **SIEM** solution.
    
- Main components:
    
    - **Elasticsearch** → Stores & searches data
        
    - **Logstash** → Collects & processes logs
        
    - **Kibana** → Visualizes data
        
    - **Beats** → Lightweight data shippers
        

**Data Flow:**

```
Beats → Logstash → Elasticsearch → Kibana
```

---

# Elastic Stack Components

### Elasticsearch

- Distributed search engine.
    
- Stores and indexes logs.
    
- Supports fast searching and analytics.
    
- Uses REST APIs.
    

### Logstash

Responsible for **ETL (Extract, Transform, Load)**.

Functions:

1. **Input** – Collect logs.
    
2. **Filter** – Parse, transform, enrich logs.
    
3. **Output** – Send logs to Elasticsearch.
    

---

### Kibana

- Visualization dashboard.
    
- Runs searches on Elasticsearch.
    
- Creates:
    
    - Dashboards
        
    - Charts
        
    - Tables
        
    - Graphs
        

> SOC analysts spend most of their time in **Kibana**.

---

### Beats

Lightweight agents that collect data.

Examples:

- **Filebeat** → Log files
    
- **Metricbeat** → System metrics
    

Can send data directly to:

- Logstash
    
- Elasticsearch
    

---

# Elastic Stack as SIEM

Used to:

- Collect security logs
    
- Store and index logs
    
- Detect threats
    
- Correlate events
    
- Visualize security data
    
- Create alerts and dashboards
    

Data sources:

- Firewalls
    
- IDS/IPS
    
- Endpoints
    
- Servers
    

---

# Kibana Query Language (KQL)

Used to search Elasticsearch data.

## Basic Syntax

```
field:value
```

Example:

```kql
event.code:4625
```

Shows Windows **failed login** events.

---

## Free Text Search

Search all indexed fields.

```kql
"svc-sql1"
```

---

## Logical Operators

- AND
    
- OR
    
- NOT
    

Example:

```kql
event.code:4625 AND user.name:admin
```

---

## Comparison Operators

- `:`
    
- `:>`
    
- `:>=`
    
- `:<`
    
- `:<=`
    
- `:!`
    

Example:

```kql
@timestamp >= "2023-03-03"
```

---

## Wildcards

Use `*`

Example:

```kql
user.name:admin*
```

Matches:

- admin
    
- administrator
    
- admin123
    

---

# Important KQL Examples

### Failed Login

```kql
event.code:4625
```

### Failed Login to Disabled Account

```kql
event.code:4625 AND winlog.event_data.SubStatus:0xC0000072
```

### Failed Login Within Time Range

```kql
event.code:4625 AND @timestamp >= "2023-03-03" AND @timestamp <= "2023-03-06"
```

---

# Finding Available Fields

### Method 1: Discover + Free Text Search

Search keywords like:

```
4625
```

or

```
0xC0000072
```

Discover reveals available fields.

---

### Method 2: Elastic Documentation

Useful references:

- Elastic Common Schema (ECS)
    
- Winlogbeat fields
    
- Filebeat fields
    

---

# Elastic Common Schema (ECS)

ECS = Standard field naming across Elastic Stack.

### Benefits

- Consistent field names
    
- Easier KQL queries
    
- Better event correlation
    
- Better dashboards
    
- Compatible with Elastic Security & ML
    
- Future-proof
    

**Prefer ECS fields** over source-specific fields whenever possible.

---

# Important Fields

| Field                         | Meaning                         |
| ----------------------------- | ------------------------------- |
| `event.code`                  | Windows Event ID (ECS)          |
| `winlog.event_id`             | Windows Event ID (Winlogbeat)   |
| `@timestamp`                  | Original event time             |
| `event.created`               | Time Elastic received the event |
| `user.name`                   | Username                        |
| `winlog.event_data.SubStatus` | Login failure reason            |

---

# Exam/Interview Points 

- **Elastic Stack = Elasticsearch + Logstash + Kibana + Beats**
    
- **Pipeline:** Beats → Logstash → Elasticsearch → Kibana
    
- **Logstash:** Input → Filter → Output
    
- **Elasticsearch:** Store, index, search
    
- **Kibana:** Dashboards & KQL searches
    
- **Beats:** Lightweight data collectors
    
- **KQL:** `field:value`
    
- **Use ECS fields** for standardized searches.
    
- **Windows Event 4625 = Failed login**
    
- **SubStatus `0xC0000072` = Account disabled**

   
    
    ## Question 1
        
    ### Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Discover". Then, click on the calendar icon, specify "last 15 years", and click on "Apply". Finally, choose the "windows*" index pattern. Now, execute the KQL query that is mentioned in the "Comparison Operators" part of this section and enter the username of the disabled account as your answer. Just the username; no need to account for the domain.	
    
    ## Solution:
    - Firstly I headed to the targeted url and opened the Discover section via side toggle.
    - ![[1.png]]
    - after that I Changed the time, indexed the window and pasted the query which was given in the section.
    - ![[2.png]]
    - Here, I got an event which I expended and after some scrolling I got the username.


- ## Question 2
        
    ### Now, execute the KQL query that is mentioned in the "Wildcards and Regular Expressions" part of this section and enter the number of returned results (hits) as your answer.
    
    ## Solution:
    - This was quite simple, I pasted the query and got the result
    - ![[3.png]]














