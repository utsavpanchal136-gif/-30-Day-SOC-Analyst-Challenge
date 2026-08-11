# 🧪 Day 02 Lab: Windows Security Logs

## Objective

Generate and investigate a Windows **Event ID 4625** failed authentication event.

---

## Requirements

- Windows 10/11
- Administrator access
- PowerShell
- Event Viewer

---

## 1. Create Test Account

Run PowerShell as Administrator:

```powershell
net user haxuser1 TestPassword123 /add
```

Verify:

```powershell
net user haxuser1
```

---

## 2. Generate Failed Login

Use an intentionally incorrect password:

```powershell
net use \\127.0.0.1\IPC$ /user:haxuser1 WrongPassword
```

Expected result:

```text
System error 1326 has occurred.

The user name or password is incorrect.
```

The failed authentication should generate a Windows Security event.

---

## 3. Locate Event 4625

Open:

```text
eventvwr.msc
```

Navigate:

```text
Windows Logs
→ Security
```

Filter:

```text
Event ID: 4625
```

---

## 4. Record the Evidence

Document:

```text
Account Name:
Account Domain:
Logon Type:
Failure Reason:
Workstation Name:
Source Network Address:
Timestamp:
```

---

## 5. Investigation

Ask:

```text
WHO?
WHAT?
WHEN?
WHERE?
WHY?
```

Then determine:

```text
Is this expected?
        ↓
Is there a pattern?
        ↓
Are there multiple failures?
        ↓
Was there a successful login afterward?
        ↓
What happened after the login?
```

---

## 6. Screenshot

Capture the Event ID 4625 details.

Save the screenshot as:

```text
screenshots/event-4625-failed-logon.png
```

---

## 7. Cleanup

After completing the lab, remove the temporary test account:

```powershell
net user haxuser1 /delete
```

Verify:

```powershell
net user haxuser1
```

The account should no longer exist.