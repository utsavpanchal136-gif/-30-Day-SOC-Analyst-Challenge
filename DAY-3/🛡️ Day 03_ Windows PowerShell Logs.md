# 🛡️ Day 03: Windows PowerShell Logs

## 🎯 What You Will Learn

Today we learn how SOC Analysts investigate **PowerShell activity** on Windows.

You will learn:

- What PowerShell logs are
- Event IDs 4103, 4104, and 4688
- How to investigate PowerShell commands
- What suspicious PowerShell patterns look like
- What LOLBins are
- How to correlate PowerShell with other events

---

# 1. What is PowerShell?

**PowerShell** is a Windows command-line and automation tool.

System administrators use it for legitimate tasks such as:

- Managing users
- Checking system information
- Automating tasks
- Managing Windows services
- Configuring computers

Example:

```powershell
Get-LocalUser
```

This lists local users on the computer.

PowerShell itself is **not malicious**.

---

# 2. Why Do Attackers Use PowerShell?

Attackers often abuse PowerShell because:

- It is already available on Windows
- It can automate tasks
- It can interact with many Windows components
- It can execute scripts
- It can communicate with other systems

This is an example of **Living off the Land**.

### 🧠 Easy Example

Imagine a thief enters a building and uses the tools already available inside instead of bringing their own tools.

Attackers can do something similar with legitimate Windows tools.

```text
Legitimate Tool
      ↓
PowerShell
      ↓
Abused by Attacker
      ↓
Malicious Activity
```

---

# 3. Important PowerShell Event IDs

| Event ID | Meaning | SOC Use |
|---|---|---|
| **4103** | PowerShell command/module activity | Investigate commands and modules |
| **4104** | PowerShell Script Block Logging | Analyze PowerShell script content |
| **4688** | Process Creation | Identify PowerShell process execution |

### 🧠 Remember

```text
4103 = COMMAND / MODULE ACTIVITY

4104 = SCRIPT CONTENT

4688 = PROCESS CREATED
```

These events can provide different pieces of the investigation.

---

# 4. Event ID 4103

```text
4103 = PowerShell Command / Module Logging
```

It can provide information about PowerShell commands and module activity.

Example:

```text
Event ID: 4103

Command:
Get-LocalUser
```

A SOC Analyst can use this to understand what PowerShell was doing.

---

# 5. Event ID 4104

```text
4104 = PowerShell Script Block Logging
```

This is useful because it can record PowerShell script content.

Example:

```powershell
Get-LocalUser
```

or:

```powershell
Get-Process
```

The analyst can examine the recorded script and determine whether it looks normal or suspicious.

---

# 6. Event ID 4688

```text
4688 = Process Creation
```

This event is generated when a new process is created, when the relevant Windows auditing is enabled.

Example:

```text
4688
Process: powershell.exe
Parent Process: explorer.exe
User: utsav
```

This can help answer:

> How was PowerShell started?

For example:

```text
explorer.exe
      ↓
powershell.exe
```

may be normal user activity.

But:

```text
winword.exe
      ↓
powershell.exe
```

could be much more interesting from a security perspective, depending on the circumstances.

This is why **parent process information matters**.

---

# 7. Enable PowerShell Logging

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

Enable the relevant logging policies:

```text
Module Logging
Script Block Logging
Script Execution
```

For this lab, the important ones are **Module Logging** and **Script Block Logging**.

---

# 🧪 8. Day 03 Lab: Generate PowerShell Activity

Open PowerShell as Administrator.

Run:

```powershell
Start-Process "notepad.exe" -ArgumentList "C:\Windows\System32\drivers\etc\hosts"
```

### What does this command do?

It starts:

```text
notepad.exe
```

and opens:

```text
C:\Windows\System32\drivers\etc\hosts
```

This is legitimate activity.

We are intentionally generating PowerShell activity so that we can investigate the resulting logs.

---

# 9. Find the PowerShell Logs

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

Look for:

```text
Event ID: 4103
Event ID: 4104
```

Record:

```text
Command
Username
Timestamp
Computer
Script/Command Content
```

📸 Take a screenshot of the relevant event.

---

# 10. Investigate the Event

Don't just find the Event ID.

Ask:

### 👤 WHO?

Who executed PowerShell?

```text
User: utsav
```

### 💻 WHERE?

Which computer generated the event?

```text
Host: DESKTOP-01
```

### 🕐 WHEN?

When did it happen?

```text
Time: 10:32:15
```

### ⚙️ WHAT?

What command was executed?

```powershell
Start-Process "notepad.exe" ...
```

### 🔗 HOW?

How was PowerShell started?

Check related process events such as **4688** when available.

### 🚨 IS IT SUSPICIOUS?

Does the command make sense in this situation?

---

# 11. Is PowerShell Malicious?

**No.**

This is one of the most important lessons.

For example:

```powershell
Get-Process
```

is a normal administrative command.

Similarly:

```powershell
Get-LocalUser
```

can be completely legitimate.

The same tools can also be abused.

Therefore:

```text
PowerShell ≠ Malware
```

Instead:

```text
PowerShell
    +
Command
    +
User
    +
Parent Process
    +
Network Activity
    +
Context
    =
Risk Assessment
```

---

# 🚨 12. Suspicious PowerShell Patterns

Some PowerShell commands or patterns deserve additional investigation.

Examples include:

```text
EncodedCommand
IEX
Invoke-WebRequest
DownloadString
FromBase64String
```

### Example suspicious chain

```text
powershell.exe
      ↓
Encoded command
      ↓
Download content
      ↓
Execute content
```

This is much more suspicious than:

```text
powershell.exe
      ↓
Get-Process
```

But even suspicious-looking commands require context before declaring a compromise.

---

# 13. What is an Encoded Command?

Attackers may encode PowerShell commands to make them harder to read.

For example:

```text
Normal command
      ↓
Encoded data
      ↓
PowerShell decodes it
      ↓
Command executes
```

Seeing an encoded PowerShell command is a **strong investigation clue**, but it is not automatically proof of malicious activity.

Some legitimate software can also use encoded commands.

---

# 🔥 14. What are LOLBins?

**LOLBin** means **Living Off the Land Binary**.

These are legitimate programs that can potentially be abused by attackers.

Examples:

| Tool | Possible Abuse |
|---|---|
| `powershell.exe` | Execute scripts |
| `certutil.exe` | Download/decode files |
| `mshta.exe` | Execute HTA content |
| `regsvr32.exe` | Execute registered DLL components |
| `rundll32.exe` | Execute DLL functions |
| `schtasks.exe` | Schedule tasks and potentially establish persistence |

### 🧠 Important

These programs are **not malware**.

The security question is:

> "Is this legitimate tool being used in a suspicious way?"

---

# 15. Real SOC Scenario

Imagine an employee opens a Word document.

The logs show:

```text
WINWORD.EXE
      ↓
PowerShell.exe
      ↓
Encoded command
      ↓
Network connection
      ↓
Suspicious file
```

This deserves investigation because the **sequence of events** is suspicious.

Compare that with:

```text
User
 ↓
PowerShell
 ↓
Get-Process
```

The second example is much more likely to be normal administration.

---

# 16. Correlation is Important

Don't investigate PowerShell in isolation.

Try connecting:

```text
4104
 ↓
PowerShell script
 ↓
4688
 ↓
Process creation
 ↓
Network logs
 ↓
EDR alerts
```

This gives the analyst a bigger picture.

### Example

```text
10:30
PowerShell starts

      ↓

10:31
Encoded command executed

      ↓

10:31
PowerShell connects to external IP

      ↓

10:32
Executable created

      ↓

10:33
Suspicious process starts
```

One event may not tell you much.

The entire sequence can tell a story.

---

# 🎯 17. SOC Challenge

After completing the lab, answer:

### 1. WHO?

Who executed PowerShell?

### 2. WHAT?

What command was executed?

### 3. WHEN?

When did it execute?

### 4. WHICH EVENT?

Was the activity recorded by:

```text
4103?
4104?
4688?
```

### 5. IS IT SUSPICIOUS?

Why or why not?

### 6. WHAT NEXT?

What additional logs would you check?

Possible sources:

```text
Windows Security Logs
PowerShell Logs
Process Creation
DNS Logs
Firewall Logs
EDR Logs
```

---

# 🧠 SOC Analyst Workflow

Use this process:

```text
PowerShell Event
       ↓
Read Command
       ↓
Identify User
       ↓
Check Host
       ↓
Check Parent Process
       ↓
Check Network Activity
       ↓
Check Related Events
       ↓
Determine Risk
       ↓
Document Finding
```

---

# 📌 Day 03 Takeaways

You should now understand:

- What PowerShell is
- Why attackers abuse PowerShell
- Event ID 4103
- Event ID 4104
- Event ID 4688
- PowerShell Script Block Logging
- Process creation logging
- Suspicious PowerShell patterns
- LOLBins
- Event correlation
- Why context matters

---

# 💼 Portfolio Skills

```text
PowerShell Log Analysis
Windows Event Analysis
Process Investigation
PowerShell Detection
LOLBin Awareness
Event Correlation
Basic Threat Detection
Security Documentation
```

---

# 🧠 Day 03 Mindset

Don't think:

> "PowerShell was executed, therefore it's malicious."

Think:

> "PowerShell was executed. What command did it run, who ran it, how was it started, what did it communicate with, and what happened afterward?"

That is the difference between **reading logs** and **investigating security events**.