# 🧪 Day 03 Lab: PowerShell Log Analysis

## Objective

Generate legitimate PowerShell activity and investigate the resulting **4103, 4104, and 4688** events.

---

## 1. Enable PowerShell Logging

Open:

```text
gpedit.msc
```

Navigate:

```text
Computer Configuration
→ Administrative Templates
→ Windows Components
→ Windows PowerShell
```

Enable:

- Module Logging
- Script Block Logging
- Script Execution

---

## 2. Generate Test Activity

Open PowerShell as Administrator:

```powershell
Start-Process "notepad.exe" -ArgumentList "C:\Windows\System32\drivers\etc\hosts"
```

Confirm that Notepad opens the `hosts` file.

---

## 3. Locate PowerShell Events

Open:

```text
eventvwr.msc
```

Navigate:

```text
Applications and Services Logs
→ Microsoft
→ Windows
→ PowerShell
→ Operational
```

Investigate:

```text
4103
4104
```

Record:

```text
User:
Computer:
Timestamp:
Command:
Script Content:
```

---

## 4. Investigate Process Creation

Navigate to:

```text
Windows Logs
→ Security
```

Search for:

```text
Event ID: 4688
```

Look for:

```text
New Process Name:
Creator Process Name:
Process Command Line:
Subject User:
Timestamp:
```

Determine whether you can establish:

```text
Parent Process
      ↓
powershell.exe
      ↓
notepad.exe
```

---

## 5. Capture Evidence

Take screenshots of the relevant events.

Save them under:

```text
screenshots/
```

Recommended names:

```text
event-4103.png
event-4104.png
event-4688.png
```

---

## 6. Investigation Questions

Answer:

```text
Who executed PowerShell?

What command was executed?

When did it execute?

Which PowerShell Event IDs were generated?

Was a 4688 event generated?

What was the parent process?

What process was created?

Is the behavior suspicious?

What additional logs would you investigate?
```

---

## 7. Cleanup

No permanent system modification is required for this test.

Close Notepad and PowerShell after collecting the evidence.