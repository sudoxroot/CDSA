
## Question 1 — DLL Hijacking (C:\Logs\DLLHijack)

The task was to figure out which process pulled off the DLL hijack.  
I knew right away to hunt for **Sysmon Event ID 7** — DLL loads.

So I dumped all ID 7 events:

Get-WinEvent -FilterHashtable @{Path='C:\Logs\DLLHijack\*.evtx'; Id=7} | Format-List

That gave me _way_ more data than any human wants to scroll through. Most of it was browser noise.

So I got more intentional and filtered for the suspicious DLL that kept popping up:

Get-WinEvent -FilterHashtable @{Path='C:\Logs\DLLHijack\*.evtx'; Id=7} |  
    Where-Object { $_.Message -like "*DismCore.dll*" } |  
    Format-List

Boom — two events, one pretending to be legit.

One was just used as the delivery vehicle.  
____.exe was the real culprit, and it showed up earlier in the timeline.

## Question 2 — Unmanaged PowerShell (Victim Process)

Folder: **C:\Logs\PowershellExec**

Goal: Find the process that executed unmanaged PowerShell.

The trick here is knowing unmanaged PowerShell makes a non-PowerShell process load **clrjit.dll** or **clr.dll,** DLLs that belong to the .NET runtime.

So I filtered for DLL loads again:

Get-WinEvent -Path "C:\Logs\PowershellExec\*.evtx" |  
    Where-Object { $_.Id -eq 7 -and $_.Message -match "clrjit.dll" } |  
    Select-Object -ExpandProperty Message

Out of everything in the logs, only **_______.exe** and PowerShell.exe loaded clrjit.dll.  
PowerShell is normal — The other is not.

## Question 3 — Who Injected into Calculator?

Same folder, now I needed the attacker, not the victim.

Process injection almost always shows up in **Sysmon Event ID 8** (CreateRemoteThread).

So I ran:

Get-WinEvent -FilterHashtable @{Path='C:\Logs\PowershellExec\*.evtx'; Id=8} | Format-List

Right there in the message:

- **SourceImage:** rundll32.exe
- **TargetImage:** Calculator.exe

Clear as day — rundll32.exe injected into Calculator.exe.

## Question 4 — Who Dumped LSASS? (C:\Logs\Dump)

LSASS dumps always trigger **Sysmon Event ID 10** (Process Access).

[](https://medium.com/write?source=promotion_paragraph---post_body_banner_home_for_stories_blocks--18dab17adaf8---------------------------------------)

I targeted lsass.exe specifically:

Get-WinEvent -FilterHashtable @{Path='C:\Logs\Dump\*.evtx'; Id=10} |  
    Where-Object { $_.Message -match "lsass.exe" } |  
    Format-List

Lots of benign entries system processes accessing LSASS is normal.

But one stood out immediately:

**___________.exe**  
launched from a user directory.

That’s not normal. That’s an attacker tool.

## Question 5 — Was There a Malicious Login After the Dump?

I extracted the timestamp of the LSASS dump (from Event ID 10), then checked if any suspicious logins happened afterward.

$dumpTime = Get-Date "2022-04-28 02:08:47Z"

Get-WinEvent -FilterHashtable @{Path='C:\Logs\Dump\*.evtx'; Id=4624} |  
    Where-Object { $_.TimeCreated -gt $dumpTime } |  
    Format-List

I saw only **Logon Type 5** (Service Logon) from **SYSTEM**, which is normal behavior.

No Logon Type 3, 9, or interactive logons.  
No lateral movement.  
Nothing malicious.

## Question 6 : Strange Parent-Child Execution (C:\Logs\StrangePPID)

Parent spoofing or “weird PPID relationships” always show up in **Sysmon Event ID 1**.

I parsed the XML cleanly (never trust property indexes):

Get-WinEvent -FilterHashtable @{Path='C:\Logs\StrangePPID\*.evtx'; Id=1} |  
    ForEach-Object {  
        $xml = [xml]$_.ToXml()  
        $image        = $xml.Event.EventData.Data | Where-Object { $_.Name -eq 'Image' } | Select-Object -ExpandProperty '#text'  
        $parentImage  = $xml.Event.EventData.Data | Where-Object { $_.Name -eq 'ParentImage' } | Select-Object -ExpandProperty '#text'  
        [PSCustomObject]@{  
            TimeCreated  = $_.TimeCreated  
            ParentImage  = $parentImage  
            Image        = $image  
        }  
    } | Format-Table -AutoSize

That gave me a crystal-clear timeline.

Most parent-child pairs looked totally normal…

…but then I saw it:

**________.exe → cmd.exe**