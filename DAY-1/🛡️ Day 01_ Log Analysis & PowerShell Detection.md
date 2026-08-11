# 🛡️ Day 01: Log Analysis & PowerShell Detection

**30-Day SOC Analyst Challenge**

## 🎯 Objective

Learn the fundamentals of security log analysis and investigate Windows PowerShell activity using **Windows Event Viewer**.

The goal of this lab is to understand how a SOC Analyst uses logs to answer:

- **Who** performed the activity?
- **What** happened?
- **When** did it happen?
- **Where** did it happen?
- **How** was it performed?
- **Is the activity suspicious?**

---

## 📚 Topics Covered

- What are logs?
- Why logs matter in cybersecurity
- Common log sources
- Windows Event Logs
- PowerShell logging
- Event ID 4104
- Basic SOC investigation methodology
- User enumeration
- Context-based detection

---

# 🔎 What is a Log?

A log is a record of an activity performed by a system, application, network device, or security tool.

Example:

```text
User: utsav
Action: PowerShell executed
Command: Get-LocalUser
Time: 10:32:15
```

A useful way to think about logs is as **CCTV footage for computer systems**.

SOC Analysts examine these records to reconstruct events and identify suspicious activity.

---

# 🛡️ Why Logs Matter

Logs can help security analysts:

- Detect attacks
- Investigate security incidents
- Identify suspicious behavior
- Track account activity
- Build an incident timeline
- Collect evidence

Attackers frequently use legitimate system tools such as PowerShell after gaining access to a Windows machine.

A simplified attack chain can look like:

```text
Initial Access
      ↓
PowerShell Execution
      ↓
System/User Enumeration
      ↓
Credential Access
      ↓
Lateral Movement
```

PowerShell activity can therefore provide useful evidence during an investigation.

---

# 🗂️ Common Log Sources

| Log Source | Example |
|---|---|
| Windows | Windows Event Logs |
| Linux | `/var/log/auth.log` |
| Firewall | Allowed/blocked connections |
| DNS | Domain lookups |
| Web Server | HTTP requests |
| Authentication | Login success/failure |
| PowerShell | Script execution |
| EDR | Endpoint activity |

Some useful Windows Event IDs include:

| Event ID | Description |
|---|---|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4104 | PowerShell Script Block Logging |

> Event IDs should always be interpreted with their surrounding context. An event by itself does not automatically mean an attack occurred.

---

# 🧪 Lab: Detecting PowerShell Activity

## Environment

**Operating System:** Windows 10/11 or Windows Server

### Tools Used

- Windows Event Viewer
- PowerShell
- Group Policy Editor

---

## Step 1: Enable PowerShell Logging

Open:

```text
gpedit.msc
```

Navigate to:

```text
Computer Configuration
→ Administrative Templates
→ Windows Components
→ Windows PowerShell
```

Enable the relevant PowerShell logging policies, including:

- Module Logging
- Script Block Logging
- Script Execution

---

## Step 2: Generate PowerShell Activity

Open PowerShell as Administrator and execute:

```powershell
Get-LocalUser | Select-Object Name, Enabled
```

This command retrieves local user accounts and their enabled/disabled status.

---

## Step 3: Investigate the Event

Open:

```text
eventvwr.msc
```

Navigate to:

```text
Applications and Services Logs
→ Microsoft
→ Windows
→ PowerShell
→ Operational
```

Filter for:

```text
Event ID: 4104
```

Search the event details for:

```text
Get-LocalUser
```

---

# 🔍 Investigation Questions

During the investigation, answer:

1. Which user executed the command?
2. What hostname generated the event?
3. What time did the activity occur?
4. What PowerShell command was executed?
5. Why could `Get-LocalUser` be useful to an attacker?
6. Is this single event enough to confirm malicious activity?

---

# 📸 Evidence

The investigation should include a screenshot of the relevant **Event ID 4104** event.

Place the screenshot here:

```text
screenshots/event-4104-powershell.png
```

### Evidence

![PowerShell Event ID 4104](screenshots/event-4104-powershell.png)

> Replace the image path above with your actual screenshot.

---

# 🚨 Analysis

The command:

```powershell
Get-LocalUser
```

enumerates local accounts on the Windows system.

This can be useful during reconnaissance because an attacker may want to identify available accounts before attempting credential access, privilege escalation, or further movement.

However, **the command itself does not prove malicious activity**.

An administrator, system administrator, security analyst, or legitimate script could execute the same command.

Therefore, additional context should be investigated.

For example:

```text
PowerShell
    ↓
Encoded Command
    ↓
Downloads File
    ↓
Executes Payload
    ↓
Creates Persistence
```

This sequence would be considerably more suspicious than a standalone `Get-LocalUser` command.

---

# 🧠 SOC Analyst Methodology

For each suspicious event, investigate:

```text
WHO?
  ↓
WHAT?
  ↓
WHEN?
  ↓
WHERE?
  ↓
HOW?
  ↓
WHY?
  ↓
IS IT SUSPICIOUS?
```

This prevents analysts from treating every unusual-looking event as a confirmed attack.

---

# 📝 Key Findings

### Finding 01

**Activity:** PowerShell executed `Get-LocalUser`

**Purpose:** Local account enumeration

**Event ID:** 4104

**Potential Security Relevance:** Could represent reconnaissance during post-exploitation.

**Verdict:** Suspicious activity cannot be confirmed from this event alone.

**Required Next Steps:**

- Investigate the executing user
- Check the process tree
- Review surrounding PowerShell events
- Review authentication events
- Look for encoded commands
- Check for downloads or network connections
- Correlate with endpoint/SIEM telemetry

---

# 🎓 Skills Learned

- Windows Event Viewer
- PowerShell logging
- Event ID analysis
- Log investigation
- User enumeration detection
- Security event correlation
- Basic SOC investigation methodology

---

# 💡 Key Lesson

> **A log is not automatically an alert. Context determines whether activity is suspicious.**

A good SOC Analyst does not simply ask:

**"Did something happen?"**

They ask:

**"What happened, who did it, why did it happen, and what happened before and after it?"**

---

# 🚀 Next Step

**Day 02 → Continue the SOC investigation journey.**