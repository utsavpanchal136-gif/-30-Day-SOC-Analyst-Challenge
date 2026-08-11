# 🛡️ Day 03: Windows PowerShell Logs

**30-Day SOC Analyst Challenge**

## 🎯 Objective

Learn how to analyze **Windows PowerShell logs** and identify potentially suspicious command execution.

This lab focuses on:

- PowerShell Event ID **4103**
- PowerShell Event ID **4104**
- Windows Event ID **4688**
- Command-line analysis
- Parent process investigation
- Suspicious PowerShell indicators
- LOLBins
- Basic event correlation

The goal is not to treat PowerShell as malicious, but to understand how attackers can abuse a legitimate Windows administration tool.

---

# 🔑 Important Event IDs

| Event ID | Description | SOC Relevance |
|---|---|---|
| **4103** | PowerShell Module Logging | Analyze PowerShell command/module activity |
| **4104** | PowerShell Script Block Logging | Analyze PowerShell script content |
| **4688** | Process Creation | Identify processes and command-line context |

### Quick Reference

```text
4103 → PowerShell command/module activity
4104 → PowerShell script content
4688 → Process creation
```

These events provide different pieces of information and can be correlated during an investigation.

---

# 🧠 Why PowerShell Matters to a SOC

PowerShell is a legitimate Windows administration and automation framework.

It is **not inherently malicious**.

However, attackers frequently abuse legitimate tools because they are already installed on Windows systems.

A simplified malicious sequence could look like:

```text
powershell.exe
      ↓
Encoded command
      ↓
Download payload
      ↓
Execute payload
      ↓
Establish persistence
```

Therefore, the important question is not:

> "Was PowerShell used?"

The better question is:

> "What did PowerShell execute, who executed it, and what happened afterward?"

---

# 🧪 Lab: Detect PowerShell Activity

## Environment

**Operating System:** Windows 10/11

### Tools

- PowerShell
- Windows Event Viewer
- Group Policy Editor
- Optional: Sysmon

---

# ⚙️ Step 1: Enable PowerShell Logging

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

Enable the appropriate PowerShell logging policies, including:

- Module Logging
- Script Block Logging
- Script Execution

---

# 💻 Step 2: Generate PowerShell Activity

Open PowerShell as Administrator.

Run:

```powershell
Start-Process "notepad.exe" -ArgumentList "C:\Windows\System32\drivers\etc\hosts"
```

This command launches Notepad and opens the Windows `hosts` file.

This is intentional benign activity for the lab.

---

# 🔎 Step 3: Investigate PowerShell Logs

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
Username
Computer
Timestamp
Command
Script Content
```

---

# 🔎 Step 4: Investigate Process Creation

Also investigate:

```text
Windows Logs
→ Security
```

Search for:

```text
Event ID: 4688
```

Look for the process created by PowerShell.

Important fields include:

```text
New Process Name
Creator Process Name
Process Command Line
Subject User
Timestamp
```

The goal is to understand the process relationship:

```text
Parent Process
      ↓
powershell.exe
      ↓
notepad.exe
```

This is called **process tree analysis**.

---

# 📸 Evidence

Capture screenshots showing the relevant events.

Save them as:

```text
screenshots/event-4103.png
screenshots/event-4104.png
screenshots/event-4688.png
```

Your screenshots should clearly show the event ID and relevant investigation fields.

---

# 🚨 SOC Perspective

PowerShell usage by itself is not an indicator of compromise.

For example:

```powershell
Start-Process "notepad.exe"
```

is normal administrative or user activity.

However, commands containing suspicious behaviors deserve additional investigation.

Examples include:

```text
EncodedCommand
FromBase64String
IEX
Invoke-WebRequest
DownloadString
```

For example:

```text
powershell.exe
      ↓
EncodedCommand
      ↓
Network connection
      ↓
Payload download
      ↓
Execution
```

This sequence is significantly more suspicious than simply launching Notepad.

---

# 🔥 LOLBins

**LOLBins** are legitimate operating-system tools that can potentially be abused by attackers for malicious purposes.

| Tool | Example Abuse |
|---|---|
| `powershell.exe` | Execute scripts |
| `certutil.exe` | File transfer/decode operations |
| `mshta.exe` | Execute HTA/script content |
| `regsvr32.exe` | Execute/register DLL content |
| `rundll32.exe` | Execute DLL functionality |
| `schtasks.exe` | Scheduled task creation/persistence |

The presence of a LOLBin does not automatically mean malicious activity.

The command, parent process, user, timing, destination, and surrounding events provide the necessary context.

---

# 🕵️ SOC Investigation Workflow

A basic investigation can follow:

```text
PowerShell Event
       ↓
Read command/script
       ↓
Identify user
       ↓
Check timestamp
       ↓
Check parent process
       ↓
Check child process
       ↓
Check network activity
       ↓
Correlate additional logs
       ↓
Benign / Suspicious / Malicious
```

---

# 🎯 SOC Challenge

Answer the following questions using your collected evidence:

1. Who executed PowerShell?
2. What command was executed?
3. When did it execute?
4. Which Event ID recorded the activity?
5. Was Event ID 4103, 4104, or both present?
6. Was a 4688 process creation event available?
7. What was the parent process?
8. What process did PowerShell create?
9. Is the command suspicious?
10. What additional logs would you investigate?

---

# 📊 Initial Assessment

Expected lab activity:

```text
PowerShell
     ↓
Start-Process
     ↓
notepad.exe
     ↓
hosts file opened
```

This behavior is expected because it was intentionally generated during the lab.

### Verdict

**Benign / Expected Lab Activity**

There is no evidence from this activity alone that the system was compromised.

---

# 🔍 What Would Make This Suspicious?

Compare the lab activity:

```text
powershell.exe
      ↓
notepad.exe
```

with a potentially suspicious sequence:

```text
powershell.exe
      ↓
EncodedCommand
      ↓
Network connection
      ↓
Download
      ↓
Execution
```

The second sequence requires significantly more investigation.

Possible additional evidence includes:

- Process creation events
- Network connections
- DNS queries
- Authentication events
- EDR telemetry
- File creation events
- Persistence-related events

---

# 🎓 Skills Learned

- PowerShell logging
- Event ID 4103
- Event ID 4104
- Event ID 4688
- Command-line analysis
- Process tree analysis
- LOLBin awareness
- Suspicious PowerShell detection
- Basic event correlation
- SOC investigation methodology

---

# 💡 Key Lesson

> **PowerShell is a legitimate administrative tool. Its presence alone does not indicate compromise.**

A SOC Analyst should investigate:

```text
Who?
What?
When?
Parent Process?
Child Process?
Network Activity?
Context?
```

before deciding whether PowerShell activity is benign, suspicious, or malicious.

---

# 🚀 Next Step

**Day 04 → Analyze Windows process creation and investigate suspicious process behavior.**