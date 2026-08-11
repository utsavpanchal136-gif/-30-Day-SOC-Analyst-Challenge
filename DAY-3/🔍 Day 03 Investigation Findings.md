# 🔍 Day 03 Investigation Findings

## Event Summary

| Field | Value |
|---|---|
| PowerShell Event | `4103 / 4104` |
| Process Event | `4688` |
| Command | `Start-Process notepad.exe` |
| User | `[YOUR USERNAME]` |
| Computer | `[YOUR HOSTNAME]` |
| Timestamp | `[YOUR TIMESTAMP]` |
| Parent Process | `[YOUR FINDING]` |
| Child Process | `notepad.exe` |

---

## Observed Activity

The following PowerShell command was executed:

```powershell
Start-Process "notepad.exe" -ArgumentList "C:\Windows\System32\drivers\etc\hosts"
```

The command launched Notepad and opened the Windows `hosts` file.

---

## Analysis

The activity was intentionally generated as part of this lab.

The PowerShell activity can be investigated through Event IDs **4103 and 4104**, while Event ID **4688** can provide additional process creation and command-line context.

The observed behavior is consistent with legitimate activity because:

- The command was intentionally executed by the analyst.
- Notepad is a legitimate Windows application.
- The `hosts` file is a legitimate Windows system file.
- No malicious payload was executed.
- No suspicious download or encoded PowerShell command was involved.

---

## Verdict

**Benign / Expected Lab Activity**

This event does not provide evidence of compromise.

---

## Suspicious Indicators to Watch For

During a real investigation, additional attention would be required if PowerShell contained:

```text
EncodedCommand
FromBase64String
IEX
Invoke-WebRequest
DownloadString
```

or if the process chain showed suspicious behavior such as:

```text
powershell.exe
      ↓
Network Connection
      ↓
File Download
      ↓
Execution
```

---

## Recommended Next Investigation

If this were a real security alert, I would correlate:

- PowerShell 4103/4104 events
- Process creation 4688
- Network connections
- DNS activity
- Authentication events
- File creation/modification
- EDR telemetry
- Persistence mechanisms

---

## Skills Learned

- PowerShell log analysis
- Event ID 4103
- Event ID 4104
- Event ID 4688
- Command-line investigation
- Process tree analysis
- LOLBin awareness
- Event correlation
- SOC triage

---

## Analyst Takeaway

**PowerShell is not the detection. The behavior is the detection.**

The analyst must determine what PowerShell executed, who executed it, how it was launched, what processes it created, and what happened afterward.