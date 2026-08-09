
# Hunt 1 – Lateral Tool Transfer to `C:\Users\Public`

### Threat Intelligence

> Stuxbot now drops tools into:

```text
C:\Users\Public
```

This maps to the MITRE ATT&CK technique **Lateral Tool Transfer**.

### What to hunt

Look for **Sysmon File Create events (Event ID 11)** where files are created under `C:\Users\Public`.

Example KQL:

```kql
event.code: 11 AND message:"C:\Users\Public*"
```


Then:

- Inspect the `file.name`
    
- Find the transferred tool whose name **starts with "r"**
    
- Open that event
    
- Submit the value of the **`user.name`** field.
    

---

# Hunt 2 – Registry Run Key Persistence

### Threat Intelligence

The malware now persists using **Registry Run Keys**.

Typical registry locations include:

```text
HKCU\Software\Microsoft\Windows\CurrentVersion\Run

HKLM\Software\Microsoft\Windows\CurrentVersion\Run
```

Sysmon logs registry modifications as **Event ID 13** (Registry Value Set).

Example KQL:


```kql
event.code:13 AND registry.path:*Run*
```

Now:

- Sort by **Oldest**
    
- Find the **first** persistence event
    
- Open it
    
- Copy the **`registry.value`** field.
    

That is the answer for Hunt 2.

---

# Hunt 3 – PowerShell Remoting

### Threat Intelligence

The attackers switched from **PsExec** to **PowerShell Remoting**.

PowerShell remoting commonly uses:

- `Invoke-Command`
    
- `Enter-PSSession`
    
- `New-PSSession`
    
- `powershell.exe`
    

Target:

```text
DC1
```

Useful KQL examples:


```kql
event.code: 4104 AND powershell.file.script_block_text: *PSSession* AND DC1
```


or simply

```kql
DC1 AND powershell*
```

Once you find the event showing remoting toward **DC1**:

- Open the event
    
- Read the **`winlog.user.name`** field
    
- Submit that value.
    

---

# Summary

|Hunt|What to Search|Expected Field|
|---|---|---|
|1|`event.code:11 AND file.path:*C:\Users\Public\*`|`user.name`|
|2|`event.code:13 AND registry.path:*CurrentVersion\Run*`|`registry.value`|
|3|PowerShell Remoting (`Invoke-Command`, `PSSession`, or `DC1`)|`winlog.user.name`|

These hunts are designed to reinforce an important threat hunting principle:

> **Start with the adversary's TTPs from the intelligence report, then translate those behaviors into searches across your telemetry.** This approach is much more resilient than searching only for fixed IOCs like hashes or IP addresses.