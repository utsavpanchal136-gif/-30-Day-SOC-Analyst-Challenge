# 🛡️ Day 01: Log Analysis & PowerShell Detection

## 🎯 What You Will Learn

Today we learn how SOC Analysts use **logs** to understand what happened on a computer.

You will learn:

- What logs are
- Why logs matter in cybersecurity
- Common log sources
- How to investigate a PowerShell event
- How to decide whether an activity is suspicious

---

# 1. What is a Log?

A **log is a record of an activity** that happened on a computer, server, application, or network.

Example:

```text
User: utsav
Host: PC-01
Action: PowerShell executed
Command: Get-LocalUser
Time: 10:32:15
```

### 🧠 Easy Way to Remember

Think of a computer's logs as **CCTV footage**.

CCTV tells you:

> Who entered → When → Where → What they did

Logs tell a SOC Analyst:

> Who acted → When → Which machine → What happened → How it happened

---

# 2. Why Are Logs Important?

Logs help a SOC Analyst:

- 🔍 Detect suspicious activity
- 🕵️ Investigate security incidents
- 👤 Track user activity
- 🚨 Find attack patterns
- 📋 Collect evidence

### Real-Life Cyber Scenario

Imagine an attacker gets access to an employee's Windows computer.

They may start investigating the machine:

```text
Compromise
    ↓
PowerShell
    ↓
Find Users
    ↓
Find System Information
    ↓
Find Credentials
    ↓
Move to Another Computer
```

The attacker may try to hide, but their actions can leave **logs behind**.

A SOC Analyst can investigate those logs.

---

# 3. Common Types of Logs

| Log Source | What It Can Tell You |
|---|---|
| Windows Event Logs | Windows activity |
| Linux Logs | Linux authentication/system activity |
| Firewall Logs | Allowed/blocked network traffic |
| DNS Logs | Domain lookups |
| Web Server Logs | Website requests |
| Authentication Logs | Login attempts |
| PowerShell Logs | PowerShell activity |
| EDR Logs | Endpoint activity |

### Example: Windows Login

```text
Event ID: 4624
Successful login
```

```text
Event ID: 4625
Failed login
```

If you see hundreds of failed logins:

```text
4625
4625
4625
4625
4625
...
```

it could indicate a **brute-force attack**.

---

# 4. What is a Log Source?

A **log source is the device, system, or application that creates logs.**

Example:

```text
Windows PC ──────┐
Linux Server ────┤
Firewall ────────┤
DNS Server ──────┼──→ SIEM → SOC Analyst
VPN ─────────────┤
EDR ─────────────┘
```

Common log sources include:

- Windows
- Linux
- Firewall
- VPN
- Router
- EDR
- Antivirus
- Cloud services
- Applications

### 🧠 Real-Life Example

A company has 500 computers.

An attacker compromises one computer.

Instead of checking 500 computers manually, the SOC can collect their logs into a **SIEM**.

The analyst can then search for suspicious activity from one place.

---

# 5. PowerShell and Cybersecurity

**PowerShell** is a legitimate Windows command-line and automation tool.

Administrators use it for:

- System administration
- Automation
- User management
- Configuration

But attackers also abuse PowerShell because it is already available on Windows.

This is commonly called **Living off the Land**.

### Example

An attacker gains access to a Windows computer and runs:

```powershell
Get-LocalUser
```

This command lists local users.

The command itself is not malicious.

But an attacker might use it during **reconnaissance** to understand the system.

---

# 6. What Does `Get-LocalUser` Do?

```powershell
Get-LocalUser
```

It displays local user accounts on a Windows machine.

Example output:

```text
Name          Enabled
----          -------
Administrator True
Guest         False
utsav         True
```

### Why might an attacker use it?

An attacker may want to know:

- Which accounts exist?
- Is the Administrator account enabled?
- Which users might be interesting?
- What accounts could potentially be targeted?

This is called **reconnaissance or enumeration**.

---

# 7. Important PowerShell Event

One useful Windows PowerShell event is:

```text
Event ID: 4104
```

### Event 4104

Event ID 4104 is associated with **PowerShell Script Block Logging**.

It can help analysts see PowerShell script content that was executed.

Example:

```text
Event ID: 4104

Command:
Get-LocalUser | Select-Object Name, Enabled
```

This gives the analyst useful information about what PowerShell was doing.

---

# 8. How a SOC Analyst Investigates a Log

Don't just look at the command.

Ask six questions:

### 👤 WHO?

Who performed the action?

```text
User: utsav
```

### 💻 WHAT?

What happened?

```text
PowerShell executed
```

### 🕐 WHEN?

When did it happen?

```text
10:32:15
```

### 🌐 WHERE?

Which computer or IP was involved?

```text
Host: PC-01
```

### ⚙️ HOW?

Which process or command caused it?

```text
powershell.exe
Get-LocalUser
```

### 🚨 IS IT SUSPICIOUS?

Does the activity make sense for this user and computer?

---

# 9. Context Is Everything

This is one of the most important SOC concepts.

Consider:

```powershell
Get-LocalUser
```

Is it malicious?

**Not necessarily.**

A system administrator may legitimately run it.

But imagine:

```text
Unknown user login
       ↓
PowerShell starts
       ↓
Get-LocalUser
       ↓
Get-Process
       ↓
Download file
       ↓
Execute suspicious file
```

Now the activity becomes much more suspicious.

### Key Lesson

> **A suspicious command does not automatically mean an attack. Context determines risk.**

---

# 10. Basic SOC Investigation Workflow

A simple SOC investigation looks like:

```text
Log/Event
   ↓
Identify User
   ↓
Identify Host
   ↓
Check Time
   ↓
Understand Command
   ↓
Check Previous/Next Events
   ↓
Look for Attack Pattern
   ↓
Determine Risk
   ↓
Document Finding
```

---

# 11. Tools Used

### Beginner Tools

**Windows**

```text
Event Viewer
PowerShell
```

**Linux**

```bash
grep
journalctl
```

### SIEM Platforms

```text
Splunk
Microsoft Sentinel
Elastic Security
Wazuh
```

### Endpoint Security

```text
Microsoft Defender
Sysmon
CrowdStrike
```

Don't try to master everything at once.

Start with:

```text
Event Viewer
      ↓
PowerShell Logs
      ↓
Linux Logs
      ↓
Wazuh / Splunk
```

---

# 🧪 Day 01 Practical Lab

## Objective

Detect PowerShell activity using Windows Event Viewer.

### Step 1: Enable PowerShell Logging

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

Enable the required PowerShell logging policies, especially:

```text
Module Logging
Script Block Logging
Script Execution
```

---

## Step 2: Generate PowerShell Activity

Open PowerShell and run:

```powershell
Get-LocalUser | Select-Object Name, Enabled
```

This generates legitimate PowerShell activity that we can investigate.

---

## Step 3: Open Event Viewer

Run:

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

Look for:

```text
Event ID: 4104
```

Search the event for:

```text
Get-LocalUser
```

---

# 🔎 Investigation Questions

When you find the event, answer:

### 1. Who?

Which user executed the command?

### 2. What?

What PowerShell command was executed?

### 3. When?

What time did it execute?

### 4. Where?

Which hostname/computer generated the event?

### 5. Why?

Why could this command be useful to an attacker?

### 6. Verdict?

Is this event alone enough to confirm an attack?

**Answer: No.**

You need additional context and related events.

---

# 🧠 Real SOC Scenario

Imagine you are working in a company's SOC.

At 10:32 AM:

```text
User: employee01
Host: FINANCE-PC-07
Process: powershell.exe
Command: Get-LocalUser
```

At first, this may be normal.

You investigate further.

Five minutes later:

```text
PowerShell
      ↓
Get-LocalUser
      ↓
Get-Process
      ↓
Network connection to unknown IP
      ↓
Suspicious executable downloaded
```

Now you have a stronger reason to investigate.

### SOC Thinking

Don't ask:

> "Is PowerShell bad?"

Ask:

> "Why was PowerShell used, by whom, on which machine, and what happened before and after it?"

That mindset is more important than memorizing Event IDs.

---

# 📌 Important Concepts

Remember these five things:

```text
LOG
 ↓
Evidence of activity

LOG SOURCE
 ↓
System creating the log

EVENT ID
 ↓
Identifies a particular Windows event

CONTEXT
 ↓
Helps determine whether activity is suspicious

INVESTIGATION
 ↓
Connect multiple events to understand what happened
```

---

# 🎯 Day 01 Takeaways

By the end of Day 01, you should understand:

- What a log is
- Why SOC Analysts need logs
- What a log source is
- Why attackers may abuse PowerShell
- What `Get-LocalUser` does
- What Event ID 4104 represents
- How to investigate a PowerShell event
- Why context matters
- How to document a basic finding

---

# 💼 Portfolio Skill

**Skill demonstrated:**

```text
Windows Log Analysis
PowerShell Detection
Event ID Analysis
Basic Incident Investigation
Security Event Documentation
```

This is the beginning of SOC work:

> **Observe → Investigate → Correlate → Decide → Document**