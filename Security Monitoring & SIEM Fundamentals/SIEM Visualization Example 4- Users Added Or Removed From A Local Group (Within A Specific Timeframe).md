

This visualization is meant to answer one SOC question:

> **"Who was added to or removed from the local Administrators group since March 5, 2023?"**

Adding users to the local **Administrators** group is a **high-risk activity** because members of this group have administrative privileges on that machine. Attackers often add accounts to this group to maintain persistence or escalate privileges.

---

# Events Used

|Event ID|Meaning|Why it matters|
|---|---|---|
|**4732**|A member was added to a security-enabled local group|Detect privilege escalation|
|**4733**|A member was removed from a security-enabled local group|Detect privilege removal or cleanup|

---

# Step 1 : Create a Visualization

Go to

```
Dashboard
→ Edit (Pencil)
→ Create Visualization
```

Choose

```
Visualization Type:
Table
```

Why Table?

Because we want to list every administrative group modification rather than show trends.

---

# Step 2 : Filter the Data

We don't want every Windows event.

We only want:

- Event 4732
    
- Event 4733
    
- Group = Administrators
    

Filter:

```text
(event.code:4732 OR event.code:4733)
AND
group.name:"Administrators"
```

### Why?

Without the filter you'll get:

- Password changes
    
- Logons
    
- Process creation
    
- Everything else
    

We're interested only in administrator group changes.

---

# Step 3 : Select the Index

Choose

```
windows*
```

Why?

Elastic stores logs in different indices.

Examples:

```
windows*
linux*
network*
firewall*
```

The events we're looking for are Windows Security Logs.

---

# Step 4 : Verify Fields

Search for

```
user.name.keyword
```

If the field exists,

Elastic knows which account performed the action.

Example:

```
Administrator
ITAdmin
HelpDesk01
SYSTEM
```

---

# Step 5 : Configure Rows

Click

```
Rows
```

Choose

```
Top Values
```

Field:

```
user.name.keyword
```

Settings:

```
Top 1000 values
Order:
Count Descending
```

---

## Why Top Values?

Elastic groups identical usernames together.

Instead of

```
Administrator
Administrator
Administrator
Administrator
```

you get

|User|Count|
|---|---|
|Administrator|58|
|ITAdmin|12|
|HelpDesk|5|

Much easier to analyze.

---

## Why `.keyword`?

Suppose the username is

```
John Smith
```

The field

```
user.name
```

is analyzed.

Elastic splits it into

```
John
Smith
```

Bad for grouping.

But

```
user.name.keyword
```

stores

```
John Smith
```

as one exact value.

That's why `.keyword` is always used for:

- Grouping
    
- Aggregations
    
- Tables
    
- Top Values
    

---

# Step 6 : Metrics

Open

```
Metrics
```

Choose

```
Count
```

This counts how many matching events exist.

Example

|User|Count|
|---|---|
|Administrator|7|

Meaning

Administrator modified the Administrators group seven times.

---

# Step 7 : Add More Rows

One column isn't enough.

Add these additional fields.

---

## Row 1

```
user.name.keyword
```

Meaning

**Who performed the action?**

Example

```
Administrator
SYSTEM
HelpDesk01
```

---

## Row 2

```
winlog.event_data.MemberSid.keyword
```

Meaning

**Which account was added or removed?**

Windows records the member by SID.

Example

```
S-1-5-21-...
```

Sometimes Elastic resolves it to a username.

Example

```
Alice
Bob
```

---

## Row 3

```
group.name.keyword
```

Meaning

Which group changed?

Should show

```
Administrators
```

Even though the filter already restricts it, displaying it lets you verify the data.

---

## Row 4

```
event.action.keyword
```

Meaning

Was the account

```
added
```

or

```
removed
```

Examples

```
member-added
member-removed
```

This immediately tells the SOC analyst what happened.

---

## Row 5

```
host.name.keyword
```

Meaning

Which computer was affected?

Example

```
DC01
SERVER01
WS-045
```

This is important because administrator group changes happen locally.

---

# Final Table

You'll end up with something like:

|User|Member|Group|Action|Host|Count|
|---|---|---|---|---|---|
|Administrator|Alice|Administrators|Added|SERVER01|1|
|HelpDesk|Bob|Administrators|Removed|WS-021|2|

This gives a complete picture:

- Who made the change
    
- Which account was affected
    
- Which group changed
    
- Whether it was an addition or removal
    
- Which machine was affected
    
- How many times it happened
    

---

# Step 8 : Restrict the Time Range

Open

```
Panel Options
→ Customize Time Range
```

Set

```
From:
March 5, 2023

To:
Now
```

This ensures the visualization only includes events from March 5, 2023 onward.

---

# Step 9 : Save

Click

```
Save and Return
```

Then save the dashboard so the visualization persists.

---

# Why This Visualization Is Valuable

From a SOC perspective, this is a high-value detection because changes to the local **Administrators** group often indicate:

- **Privilege escalation** by an attacker after gaining access.
    
- **Persistence**, where an attacker adds a backdoor account to retain administrative access.
    
- **Unauthorized administrative activity** by insiders or compromised accounts.
    
- **Legitimate IT administration**, which can be reviewed and validated against change records.
    

Monitoring these events helps quickly identify potentially dangerous changes to local administrative privileges before they are abused.





  

## Question 1

### Navigate to http://[Target IP]:5601, click on the side navigation toggle, and click on "Dashboard". Extend the visualization we created or the "User added or removed from a local group" visualization, if it is available, and enter the common date on which all returned events took place as your answer. Answer format: 20XX-0X-0X

## Solution:
- Firstly I opened the Dashboard and opened the mentioned visualization.
- There I saw the timestamp already mentioned.
  <img width="1889" height="328" alt="Screenshot 2026-07-04 092621" src="https://github.com/user-attachments/assets/04d619bc-38d6-4aac-ab0d-29bbb08b1716" />



