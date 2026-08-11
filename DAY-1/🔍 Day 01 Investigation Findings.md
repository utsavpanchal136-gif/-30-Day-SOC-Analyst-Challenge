# 🔍 Day 01 Investigation Findings

## Incident Summary

A PowerShell command was executed to enumerate local Windows user accounts.

## Observed Activity

```text
Command:
Get-LocalUser | Select-Object Name, Enabled

Event ID:
4104

Log Source:
Microsoft-Windows-PowerShell/Operational
```

## Investigation

| Question | Finding |
|---|---|
| Who? | `[YOUR USERNAME]` |
| What? | PowerShell user enumeration |
| When? | `[DATE / TIME]` |
| Where? | `[HOSTNAME]` |
| How? | `powershell.exe` |
| Why? | Local account enumeration |

## Security Relevance

Local account enumeration can be used during reconnaissance after an attacker gains access to a Windows system.

However, this activity can also be completely legitimate.

## Verdict

**Not enough evidence to confirm malicious activity.**

Additional telemetry should be reviewed before classifying this as an incident.

## Recommended Investigation

- Review PowerShell events before and after Event ID 4104.
- Check the PowerShell process tree.
- Investigate the executing account.
- Review authentication events such as 4624 and 4625.
- Look for encoded PowerShell commands.
- Investigate unusual network connections.
- Correlate the activity with EDR or SIEM data.

## Analyst Takeaway

The most important lesson from this investigation is that **context matters**.

A single PowerShell event should not automatically be classified as malicious.