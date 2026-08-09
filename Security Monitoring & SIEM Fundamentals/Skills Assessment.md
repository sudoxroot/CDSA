
The following table summarize the whole information given in the data: 

| Visualization                            | What to Look For                         | Recommended Action                                           |
| ---------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| **1. Failed Logons (All Users)**         | Repeated failures, especially `sql-svc1` | Investigate; consult IT if needed                            |
| **2. Disabled User Logon**               | `Anni` attempted login                   | Investigate source; consult IT                               |
| **3. Failed Admin Logons**               | Admin logins not from PAW/DC             | Investigate; escalate if unauthorized                        |
| **4. RDP Service Account**               | Service account used for RDP             | High priority; likely escalate                               |
| **5. Local Administrators Group Change** | User added/removed from Administrators   | Consult IT first; escalate if unauthorized                   |
| **6. Admin Logon Not from PAW**          | Admin using non-PAW system               | Consult IT first; escalate if unauthorized                   |
| **7. SSH Root Login**                    | Any successful remote `root` login       | High priority; immediate investigation and likely escalation |
## **Visualization 1: Failed logon attempts (All users)**

> Question : Navigate to Target , click on the side navigation toggle, and click on “Dashboard”. Review the “Failed logon attempts [All users]” visualization of the “SOC-Alerts” dashboard. Choose one of the following as your answer: “Nothing suspicious”, “Consult with IT Operations”, “Escalate to a Tier 2/3 analyst”

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*TxylQQ2vseHZXd80VTRSWw.png)

Failed Logon attempts(All Users)

The above image is the Failed Logon attempts (All users), let’s unfold the investigation like a SOC analyst. First thing first, remember the Logon Type and what it mean to be and, prefer to the case scenario which is given above.

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*ySUVPTjS6pTsuEhMxqB18Q.png)

Question and Answer

**Explanation of the answer:**

1. **PAW is Logging Network Activity on DC2**

- The Privileged Admin Workstation (PAW) should be used by IT administrators for all privileged tasks.
- However, PAW is logging a high number of network logins (4 logins) from DC2.

**Suspicious?**

- If PAW is intended for admins, it should not be performing network logins from DC2.
- This might indicate misuse or an unexpected process running from PAW

**2. “Administrator” & “administrator” Accounts Are Frequently Used**

- We know from IT policy that the IT team should not be using default administrator accounts.
- Yet, both “Administrator” and “administrator” accounts show:
- Multiple Interactive Logins (3)
- Unlocking activity
- Logins on PAW and DC2

**Suspicious?**

- IT admins might still be using these accounts instead of their named accounts.
- If these logins were not expected, this could indicate unauthorized access.

**3. Service Account (sql-svc1) Performing Network Logins**

- Service accounts should not be used interactively.
- sql-svc1 shows network logins on PKI.

**Suspicious?**

- If sql-svc1 was not expected to perform network logins, this could indicate compromise or abuse.

**4. Unexpected Activity from eAdministrator**

- The eAdministrator account logged in via Network on DC1.

**Suspicious?**

- If eAdministrator is a regular admin, this could be fine.
- However, if this was an unexpected login, it needs to be checked.

### Justification

1. The PAW workstation should not be logging network logins on DC2. Needs verification.
2. The Administrator accounts are being used despite best practice policies against it.
3. Service account (sql-svc1) network activity should be confirmed as expected.
4. eAdministrator login from DC1 should be verified.
5. No immediate evidence of an attack, so escalation to Tier 2/3 is not yet necessary.

## `Visualization 2: Failed logon attempts (Disabled user)`

> Question :Navigate to Target , click on the side navigation toggle, and click on “Dashboard”. Review the “Failed logon attempts [Disabled user]” visualization of the “SOC-Alerts” dashboard. Choose one of the following as your answer: “Nothing suspicious”, “Consult with IT Operations”, “Escalate to a Tier 2/3 analyst”

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*cKMC4mlDczAqWY4t4GWWFg.png)

Failed Logon attempts (Disabled user)

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*B-lApWqe-e7GFI5RMr4Dhw.png)

Question and Answer

**Explanation of the answer:**

1. **WS001 Is a Workstation, Not a Server**

- The log source WS001 suggests that this is an end-user workstation rather than a server.

**Suspicious?**

- If WS001 was previously assigned to anni, it might still have cached credentials.
- If anni was not supposed to be logging in, someone may have attempted to use this account intentionally.

### Justification

1. A disabled account should not be attempting logins, IT should verify if this is expected.
2. A single failed attempt suggests a possible auto-login process or outdated credentials.
3. WS001 should be checked for any stored credentials or scheduled tasks using ‘anni’.
4. **Evidence of unauthorized access attempt which leads to escalation to Tier 2/3.**

## Visualization 3: Failed logon attempts (Admin users only)

> Question :Navigate to Target, click on the side navigation toggle, and click on “Dashboard”. Review the “Failed logon attempts [Admin users only]” visualization of the “SOC-Alerts” dashboard. Choose one of the following as your answer: “Nothing suspicious”, “Consult with IT Operations”, “Escalate to a Tier 2/3 analyst”

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*xDJazwIXGoveQxYCouqFeA.png)

Failed logon attempts (Admin users only)

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*fS8JCEMpD-Ed3MFlXG_hIA.png)

Question and Answer

**Explanation of the answer:**

1. Failed Logons with “Administrator” and “administrator” Accounts but a small number of failed attempts can happen due to mistyped passwords or session timeouts.
2. Unlock Attempts on DC1 and PAW usually occur when a user enters the wrong password after a session lock, If an authorized admin simply mistyped their password while unlocking their workstation, this is not unusual.

### Justification

1. Small number of failed logins likely due to normal admin activity.
2. Logon attempts occurred on expected machines (DC1, PAW, DC2).
3. No excessive failures, suggesting no brute-force attack.
4. Unlock failures are common when admins mistype their passwords.
5. If failures became frequent or appeared from unexpected locations, further investigation would be needed.

## `Visualization 4: RDP logon for service account`

> Question :Navigate to Target, click on the side navigation toggle, and click on “Dashboard”. Review the “RDP logon for service account” visualization of the “SOC-Alerts” dashboard. Choose one of the following as your answer: “Nothing suspicious”, “Consult with IT Operations”, “Escalate to a Tier 2/3 analyst”

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*5apo92SJjuXaTdJnDWf_HQ.png)

`RDP logon for service account`

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*HjoDbyu2JpDcMhcb12McBQ.png)

Question and Answer

**Explanation of the answer:**


**1. Service Account (sql-svc1) Used for RDP Login**

- Service accounts are typically designed to run background processes, not for interactive logins.
- sql-svc1 was used to establish two RDP logins to PKI from 192.168.28.130.

**Suspicious?**

- If sql-svc1 is strictly meant for SQL services and not for remote desktop access, this is highly unusual.
- Service accounts should have restricted permissions and should never be used for RDP sessions.
- If this login was unexpected, it could indicate credential theft or misuse.

**2. Source IP (192.168.28.130) Needs Verification**

- Where is `192.168.28.130` located?
- If this IP belongs to a known and authorized IT administrator machine, the activity may be expected.
- If the IP is unknown, this could indicate a compromised machine or lateral movement attempt.

**3. Unusual Privilege Escalation Risk**

- If an attacker gained access to sql-svc1, they could leverage its permissions to escalate privileges.

**Suspicious?**

- If sql-svc1 was not supposed to have RDP access, this could mean someone is misusing the account.
- An attacker might be moving laterally to access PKI, which is a critical security infrastructure component.

### Justification

1. Service accounts should not be used for RDP sessions possible compromise or misuse.
2. PKI is a high-value system, making this an important security concern.
3. The source IP (192.168.28.130) needs further investigation is it an admin machine or an unknown system?
4. If unauthorized, this could indicate credential theft or lateral movement.

**Due to the risk of privilege escalation, this incident should be escalated immediately.**

## `Visualization 5: User added or removed from a local group`

> Question :Navigate to Target, click on the side navigation toggle, and click on “Dashboard”. Review the “User added or removed from a local group” visualization of the “SOC-Alerts” dashboard. Choose one of the following as your answer: “Nothing suspicious”, “Consult with IT Operations”, “Escalate to a Tier 2/3 analyst”

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*IqemAZHlgd0-15sSikwl5g.png)

`User added or removed from a local group`

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*c8l9yTW4N4iih_7K2tLJPw.png)

Question and Answer

**Explanation of the answer:**

Administrator is the one who is performing this task and the count is only 1, the main thing we have to look for is the user have SID(user which is added to group) only without any username.

### Justification

1. As this task performed by the Administrator that’s why you should have to consult with IT operation and after confirming from them we should have to take an actionable step.

## `Visualization 6: SSH Logins`

> Question :Navigate to Target, click on the side navigation toggle, and click on “Dashboard”. Review the “SSH Logins” visualization of the “SOC-Alerts” dashboard. Choose one of the following as your answer: “Nothing suspicious”, “Consult with IT Operations”, “Escalate to a Tier 2/3 analyst”


Tribute to: https://medium.com/@k0schei/understanding-dashboard-of-elk-and-critical-thinking-like-soc-analyst-part-3-76e706955a6f

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*4AHaKG24cKtvuDgr8cll9w.png)

`SSH Logins`

Press enter or click to view image in full size

![](https://miro.medium.com/v2/resize:fit:700/1*pfGvl7UueIpXa_ns5u-QMw.png)

Question and Answer

**Explanation of the answer:**

1. The outcome of an action is failed and the user is on root trying to do ssh_login. The login attempts is not so acceptable.

**2. Source IP (192.168.28.150) Needs Verification**

- Where is `192.168.28.150` located?
- If this IP belongs to a known and authorized IT administrator machine, the activity may be expected.
- If the IP is unknown, this could indicate a compromised machine or lateral movement attempt.

### Justification

Looking at the login counts which is 6 that is not acceptable for the root user that why we should escalate a Tier 2/3 analyst.

### Tribute to:
https://medium.com/@k0schei/understanding-dashboard-of-elk-and-critical-thinking-like-soc-analyst-part-3-76e706955a6f
